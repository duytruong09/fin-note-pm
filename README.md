# FIN-NOTE Project Manager

Monorepo quản lý toàn bộ project Fin-Note - ứng dụng quản lý chi tiêu bằng giọng nói.

## 📁 Project Structure

```
fin-note-pm/
├── fin-note-app/          # Frontend - React Native + Expo
├── fin-note-be/           # Backend - NestJS + PostgreSQL
├── docs/                  # Documentation
└── .prompt/              # AI Prompts
```

## 🚀 Quick Start

### Clone Repository với Submodules

```bash
# Clone repo chính kèm tất cả submodules
git clone --recursive https://github.com/duytruong09/fin-note-pm.git

# Hoặc nếu đã clone rồi, pull submodules:
git submodule update --init --recursive
```

### Setup Backend

```bash
cd fin-note-be
npm install
docker-compose up -d
npx prisma migrate dev
npm run start:dev
```

### Setup Frontend

```bash
cd fin-note-app
npm install
npx expo start
```

## 🔄 Working with Submodules

### Pull changes từ tất cả submodules

```bash
git submodule update --remote --merge
```

### Commit changes trong submodule

```bash
# Vào trong submodule
cd fin-note-app

# Commit và push như bình thường
git add .
git commit -m "feat: add new feature"
git push origin master

# Quay lại repo chính và update submodule reference
cd ..
git add fin-note-app
git commit -m "chore: update fin-note-app submodule"
git push origin master
```

### Clone submodule mới khi teammate pull

```bash
# Khi có submodule mới được thêm
git pull
git submodule update --init --recursive
```

## 📖 Documentation

Xem thêm tài liệu chi tiết tại:
- [Backend Documentation](./fin-note-be/README.md)
- [Frontend Documentation](./fin-note-app/README.md)
- [Project Guidelines](./.claude/CLAUDE.md)

## 🛠️ Tech Stack

- **Backend**: NestJS, TypeScript, PostgreSQL, Prisma ORM
- **Frontend**: React Native, Expo, TypeScript
- **AI**: OpenAI Whisper (STT) + GPT-4o-mini (parsing)

## 📝 License

MIT
