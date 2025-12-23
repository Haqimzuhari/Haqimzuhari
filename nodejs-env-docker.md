# 🚀 Node.js Workspace with Docker

Minimal docker-compose to spin up a Node.js environment where you can create React or Express projects from inside the container.

---

## 📝 docker-compose.yml

```yaml
services:
  node:
    image: node:24-alpine
    container_name: node_workspace
    working_dir: /app
    volumes:
      - ./:/app
    stdin_open: true
    tty: true
    command: sh
```

---

## 🚀 Usage

### Start the Container

```bash
# 1. Create docker-compose.yml in your project folder
# 2. Start container
docker-compose up -d

# 3. Access container
docker-compose exec node sh
```

### Inside Container - Create React App

```bash
# You're now inside the container at /app

# Create React app
npx create-react-app .

# Or create in subfolder
npx create-react-app my-react-app

# Exit
exit
```

### Inside Container - Create Express App

```bash
# You're now inside the container at /app

# Initialize npm
npm init -y

# Install Express
npm install express nodemon

# Create folder
mkdir src

# Exit
exit
```

### Stop Container

```bash
docker-compose down
```

---

## 💡 Quick Commands

```bash
# Start workspace
docker-compose up -d

# Enter container
docker-compose exec node sh

# Stop workspace
docker-compose down
```

---

**Happy Coding! 🚀**