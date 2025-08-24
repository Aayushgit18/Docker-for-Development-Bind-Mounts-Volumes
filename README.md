# Docker-for-Development-Bind-Mounts-Volumes

# 🚀 Docker for Development: Bind Mounts + Volumes

When developing with Docker, changes to your local files are not reflected inside the container **unless you rebuild the image**. This is inefficient during development.

### ✅ Solution: Use **Bind Mounts** to sync local files with the container in real-time.

---

## 📦 Step-by-Step Guide

---

## 🛠 1. Dockerfile (Simple Setup)

```Dockerfile
# Use a base image
FROM node:18

# Set working directory inside container
WORKDIR /app

# Copy everything from local project into container
COPY . .

# Install dependencies
RUN npm install

# Run the server
CMD ["node", "server.js"]
```

---

## 🏗 2. Build the Image

```bash
docker build -t feedback-app .
```

---

## 🚀 3. Run Container with Named Volume + Bind Mount + Anonymous Volume

### 🔧 Syntax

```bash
docker run -d -p 3000:80 --name <container_name> \
-v <named_volume_name>:/app/feedback \
-v "<absolute_path_to_local_project_folder>":/app \
-v /app/node_modules \
feedback-app
```

### 📘 Explanation of Volume Flags

| Part                                   | Explanation                                                                                                      |
| -------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| `-v <named_volume_name>:/app/feedback` | A **named volume** used to persist user-submitted feedback data (e.g. files saved by app). Stored by Docker.     |
| `-v "<absolute_path>":/app`            | A **bind mount** to sync your **local project folder** with `/app` inside the container. Ensures live reloading. |
| `-v /app/node_modules`                 | An **anonymous volume** to prevent `node_modules` from being overwritten by the bind mount.                      |

> 💡 Docker prioritizes **more specific paths**. So `/app/node_modules` (anonymous volume) **overwrites** the empty `node_modules` folder from your bind mount — preserving the installed dependencies from the image.

---

### ✅ Full Example (macOS/Linux)

```bash
docker run -d -p 3000:80 --name feedback-container \
-v feedback:/app/feedback \
-v "$(pwd)":/app \
-v /app/node_modules \
feedback-app
```

### ✅ Full Example (Windows CMD)

```cmd
docker run -d -p 3000:80 --name feedback-container ^
-v feedback:/app/feedback ^
-v "%cd%":/app ^
-v /app/node_modules ^
feedback-app
```

---

## 🪵 4. Debugging: Container Crashes? View Logs

```bash
docker logs <container_name>
```

**Example:**

```bash
docker logs feedback-container
```

🔴 If you see:

```
Error: Cannot find module 'express'
```

It means the `node_modules` folder was overwritten by the bind mount, and your **local folder doesn't have the dependencies installed**.

---

## ✅ Solution Recap

| Problem                                                                                | Fix                                                             |
| -------------------------------------------------------------------------------------- | --------------------------------------------------------------- |
| Local `node_modules` doesn't exist and overrides the one in the image (via bind mount) | Add **anonymous volume** `-v /app/node_modules` to protect it   |
| Container crashes on start                                                             | Check with `docker logs`, and inspect bind mount paths          |
| Local file changes not reflected                                                       | Ensure bind mount points to project folder (`-v "$(pwd)":/app`) |

---

## 💡 What Are Anonymous Volumes?

Anonymous volumes are volumes **without a name**, declared like this:

```bash
-v /container/path
```

* Docker manages them
* They're removed with the container (unless persistent)
* Great for folders like `node_modules` that should not be overridden by bind mounts

---

## 🧪 Test the Setup

1. Run container with all volumes (as above)
2. Visit `localhost:3000`
3. Edit an HTML or JS file in your project
4. Refresh browser — your changes should show instantly without rebuilding

---

## 🔁 Common Volume Path Reference

| OS            | Shortcut           |
| ------------- | ------------------ |
| macOS / Linux | `-v "$(pwd)":/app` |
| Windows (CMD) | `-v "%cd%":/app`   |

---

## 📂 Docker Desktop: Folder Sharing (Important!)

If using **Docker Desktop**, make sure your project folder is shared:

* Docker Desktop → ⚙️ Settings → **Resources → File Sharing**
* Add your project’s parent folder if it’s not listed

---

## ✅ Summary: Full Command Breakdown

```bash
docker run -d -p 3000:80 --name feedback-container \
-v feedback:/app/feedback \         # Named volume for persistent data (e.g. user feedback)
-v "$(pwd)":/app \                  # Bind mount for live code syncing (local to container)
-v /app/node_modules \              # Anonymous volume to preserve node_modules
feedback-app                        # Image name

____________________________________________________________________________________________________________________________________________________________________________
## 🧩 THE PROBLEM (Without Anonymous Volume)

You're running this container:

```bash
docker run -d -p 3000:80 --name feedback-container \
-v "$(pwd)":/app \
feedback-app
```

### 🔥 What Happens Here?

* `-v "$(pwd)":/app` → You're **replacing everything** in `/app` inside the container with your **local project folder**.
* That includes:

  * Your `.js`, `.html`, etc.
  * But **NOT** your `node_modules`, because your local system probably doesn't have it (you never ran `npm install` locally).

### 💥 RESULT:

* Your **bind mount overwrites** the entire `/app` folder inside the container, **including deleting the `node_modules` folder** that was created when the image was built.
* So now, when your app tries to run, it says:

```
Error: Cannot find module 'express'
```

Because that dependency was installed in `/app/node_modules`, and your mount just wiped it out!

---

## ✅ THE SOLUTION: Add Anonymous Volume for `node_modules`

We fix this by **protecting `/app/node_modules`** using another volume:

```bash
-v /app/node_modules
```

### 🔍 Why This Works:

Docker uses a rule:

> **"The more specific volume path wins."**

So in our case:

| Mount Path          | Type             | Priority |
| ------------------- | ---------------- | -------- |
| `/app`              | Bind mount       | Lower    |
| `/app/node_modules` | Anonymous volume | Higher   |

That means:

* Your code (`server.js`, `index.html`, etc.) comes from your local folder via `/app`
* But the `node_modules` folder comes from Docker’s internal volume — it **survives** and is **not overwritten** by your local empty folder

---

## 🧪 Real Example

### 🔴 Without Anonymous Volume

```bash
docker run -d -p 3000:80 --name feedback-app \
-v "$(pwd)":/app \
feedback-app
```

> 💥 Crashes with: `Cannot find module 'express'`

---

### ✅ With Anonymous Volume

```bash
docker run -d -p 3000:80 --name feedback-app \
-v "$(pwd)":/app \
-v /app/node_modules \
feedback-app
```

> ✅ Works! Because `node_modules` is protected by Docker's internal volume.

---

## 🔁 Recap: Why Add Anonymous Volume?

| 🔍 Issue                                 | ❌ Without Anonymous Volume | ✅ With Anonymous Volume                        |
| ---------------------------------------- | -------------------------- | ---------------------------------------------- |
| `node_modules` exists after image build? | Yes                        | Yes                                            |
| Bind mount replaces `/app`?              | Yes                        | Yes                                            |
| `node_modules` overwritten?              | Yes                        | ❌ No (protected by `/app/node_modules` volume) |
| App crashes?                             | Yes                        | ❌ No                                           |

---

## 💡 Anonymous Volume: Simple Definition

```bash
-v /app/node_modules
```

* No name assigned (anonymous)
* Docker manages it
* Automatically created
* Ensures contents (like `node_modules`) are not lost due to other mounts

---

## ✅ Final Working Command

```bash
docker run -d -p 3000:80 --name feedback-container \
-v feedback:/app/feedback \
-v "$(pwd)":/app \
-v /app/node_modules \
feedback-app
```

