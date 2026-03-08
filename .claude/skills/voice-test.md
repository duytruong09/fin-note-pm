---
name: voice-test
description: Test the voice-to-expense pipeline end-to-end
tags: [testing, voice, ai]
---

# Voice Pipeline Testing

This skill helps you test the complete voice processing pipeline.

## What it does

1. Checks if backend server is running
2. Tests Whisper API connection
3. Tests GPT parsing with sample transcripts
4. Validates the complete pipeline
5. Shows performance metrics

## Usage

```bash
/voice-test [language]
```

**Parameters:**
- `language` (optional): `vi` or `en` (default: `vi`)

## Test Cases

### Vietnamese Test Cases

1. **Basic expense**
   - Input: "Hôm nay ăn cơm 10 nghìn"
   - Expected: {amount: 10000, category: "Food", date: today}

2. **With payment method**
   - Input: "Hôm qua đổ xăng 200k bằng thẻ"
   - Expected: {amount: 200000, category: "Transport", paymentMethod: "card"}

3. **Large amount**
   - Input: "Mua điện thoại 15 triệu"
   - Expected: {amount: 15000000, category: "Shopping"}

4. **Missing amount (low confidence)**
   - Input: "Ăn phở"
   - Expected: {confidence: < 0.5}

### English Test Cases

1. **Basic expense**
   - Input: "Had lunch today for 10 dollars"
   - Expected: {amount: 10, category: "Food", date: today}

2. **With payment method**
   - Input: "Filled up gas yesterday 50 bucks with card"
   - Expected: {amount: 50, category: "Transport", paymentMethod: "card"}

## Implementation

When this skill is invoked:

1. **Check backend health**
   ```bash
   curl http://localhost:3000/health
   ```

2. **Test Whisper API** (optional, requires audio file)
   - Upload sample audio file
   - Verify transcript accuracy

3. **Test GPT Parser**
   ```bash
   curl -X POST http://localhost:3000/api/v1/voice/test-parse \
     -H "Content-Type: application/json" \
     -d '{
       "transcript": "Hôm nay ăn cơm 10 nghìn",
       "language": "vi"
     }'
   ```

4. **Run all test cases**
   - Execute each test case
   - Compare actual vs expected output
   - Calculate accuracy metrics

5. **Display results**
   ```
   ✅ 8/10 tests passed (80%)
   ⚠️  2 tests with warnings
   ❌ 0 tests failed

   Performance:
   - Average parsing time: 1.2s
   - Average confidence: 0.87
   - Category accuracy: 90%
   ```

## Success Criteria

- ✅ Backend is running
- ✅ OpenAI API key is configured
- ✅ Parsing success rate > 90%
- ✅ Average confidence > 0.8
- ✅ Response time < 3s

## Troubleshooting

### Backend not running
```bash
cd fin-note-be
npm run start:dev
```

### OpenAI API key not set
```bash
# Check .env file
cat .env | grep OPENAI_API_KEY
```

### Low parsing accuracy
- Review failed test cases
- Check `.prompt/expense-parser-*.txt`
- Update prompt with failing examples
- Re-run tests

## Example Output

```
🧪 Testing Voice Pipeline (Vietnamese)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

✅ Backend health check passed
✅ OpenAI API connection successful

Running test cases...

[1/10] ✅ Basic expense
  Input: "Hôm nay ăn cơm 10 nghìn"
  Output: {amount: 10000, category: "Food", confidence: 0.95}
  Time: 1.1s

[2/10] ✅ With payment method
  Input: "Hôm qua đổ xăng 200k bằng thẻ"
  Output: {amount: 200000, category: "Transport", paymentMethod: "card"}
  Time: 1.3s

[3/10] ⚠️  Missing amount
  Input: "Ăn phở"
  Output: {amount: 0, category: "Food", confidence: 0.45}
  Warning: Low confidence (expected < 0.5)
  Time: 0.9s

...

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Summary:
  ✅ 8 passed
  ⚠️  2 warnings
  ❌ 0 failed

Metrics:
  Success rate: 100%
  Avg confidence: 0.87
  Avg response time: 1.2s
  Category accuracy: 100%

✨ All tests passed!
```
