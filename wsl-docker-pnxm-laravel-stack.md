# 🚀 Docker-Laravel Stack 🐳

This guide will help you set up a Laravel development environment using Docker, WSL, and VS Code Remote. 🎯

---   

## 📌 Prerequisites

✅ Ensure you have **WSL** and **Docker** installed. And also ready to clone `docker-pnxm`. Follow the guides below:
  - 📄 [WSL-Docker Setup](./wsl-docker-stack.md)
  - 📄 [Docker-PNXM Guide](https://github.com/Haqimzuhari/docker-pnxm)

## ⚙️ Setting Up the Environment

### 1️⃣ Remote WSL Setup
🖥️ Open **VS Code** and install the **Remote - WSL** extension.
🔹 Open a **WSL terminal** and navigate to your project directory.

### 2️⃣ Configure `docker-compose` ⚡
📂 Clone your project repository or create a new one.
📝 Add a `docker-compose.yml` file to configure Laravel, MySQL, and other services.

### 3️⃣ Start Docker Services 🏗️
```sh
docker-compose up -d
```
✅ Your Docker services should now be running!

### 4️⃣ Connect to the App Container 🛠️
🔹 Open VS Code Command Palette (`Ctrl + Shift + P`)
🔹 Select **Remote-Containers: Attach to Running Container**
🔹 Choose the **app** container

### 5️⃣ Create a Laravel Project 🏗️
Inside the container terminal:
```sh
composer create-project --prefer-dist laravel/laravel .
```
📌 Laravel project successfully created!

### 6️⃣ Configure `.env` for Database Connection 🛢️
📝 Update the `.env` file with the correct database credentials:
```
DB_CONNECTION=mysql
DB_HOST=db
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=root
DB_PASSWORD=root
```

### 7️⃣ Install Dependencies 📦
```sh
composer install
npm install
php artisan migrate
```
✅ Your Laravel project is now set up with all dependencies!

### 8️⃣ Open the Laravel Project in Browser 🌐
```sh
php artisan serve --host=0.0.0.0 --port=8000
```
🚀 Open `http://localhost:8000` in your browser to see your Laravel app in action!

### 9️⃣ Connect to Database in DBeaver 🛢️
- Open **DBeaver** 🖥️
- Connect to **MySQL** using:
  - **Host:** `localhost`
  - **Port:** `3306`
  - **User:** `root`
  - **Password:** `root`
  - **Database:** `laravel`

## 🎉 Done! 🚀
Your Laravel project is now running inside a Docker container with a proper development setup. Happy coding! 💻🔥
