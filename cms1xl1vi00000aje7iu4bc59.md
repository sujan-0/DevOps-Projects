---
title: "Multistage Docker Builds & Distroless Images: How I Shrunk My Docker Image by 800%"
datePublished: 2026-07-26T15:06:31.254Z
cuid: cms1xl1vi00000aje7iu4bc59
slug: multistage-docker-builds-distroless-images-how-i-shrunk-my-docker-image-by-800

---

If you've ever built a Docker image and been shocked to see it's **hundreds of megabytes** just to run a tiny app, this note is for you. Today I'll explain two simple ideas — **multistage builds** and **distroless (minimal) images** — that helped me cut my image size by a huge margin, using a simple Go calculator app as an example.

By the end of this note, you'll understand:

*   Why normal Docker images are so big
    
*   What a multistage build is, and how to write one
    
*   What a "distroless" or minimal base image is
    
*   Why smaller images actually matter in real projects
    

### 1\. The Problem: Why Docker Images Get Huge

When you build a Docker image the "normal" way, you usually start from a full operating system image like `ubuntu`, then install all the tools you need to **build** your app (compilers, package managers, libraries) inside that same image.

The catch: Docker doesn't forget those build tools once your app is built. They stay in the final image forever, even though you only needed them for a few minutes during the build. So you end up shipping:

*   A full Linux OS
    
*   Compilers and build tools (like `golang-go`)
    
*   Your actual app (which is usually tiny in comparison)
    

That's like packing your entire garage full of tools just to hand someone a single hammer.

### 2\. Example: A Normal (Single-Stage) Dockerfile

Here's a simple Dockerfile for a Go calculator app, built the normal way:

```dockerfile
###########################################
# BASE IMAGE
###########################################

FROM ubuntu AS build

RUN apt-get update && apt-get install -y golang-go

ENV GO111MODULE=off

COPY . .

RUN CGO_ENABLED=0 go build -o /app .

ENTRYPOINT ["/app"]
```

Build it with:

```bash
docker build -t simplecalculator .
```

This works fine, but the final image contains the entire Ubuntu OS plus the Go compiler — none of which your app needs to actually *run*. That's why the image size ends up large (often 700MB or more for something this simple).

### 3\. What is a Multistage Docker Build?

A **multistage build** lets you use one stage of the Dockerfile to **build** your app, and a separate, much smaller stage to **run** it. You copy over only the final compiled file (the binary) into a clean, empty image — and leave everything else behind.

Think of it like cooking in a messy kitchen, then serving the finished dish on a clean plate. The mess (compilers, package managers, source code) stays in the kitchen. Only the final dish goes to the table.

Here's the same app, rewritten as a multistage build:

```dockerfile
###########################################
# BASE IMAGE (build stage)
###########################################

FROM ubuntu AS build

RUN apt-get update && apt-get install -y golang-go

ENV GO111MODULE=off

COPY . .

RUN CGO_ENABLED=0 go build -o /app .

############################################
# HERE STARTS THE MAGIC OF MULTI STAGE BUILD
############################################

FROM scratch

# Copy the compiled binary from the build stage
COPY --from=build /app /app

# Set the entrypoint for the container to run the binary
ENTRYPOINT ["/app"]
```

Build it with:

```bash
docker build -t simplecalculator-multistage .
```

**What changed?**

*   The first `FROM ubuntu AS build` stage is only used to compile the Go program. It's given a name (`build`) so we can refer to it later.
    
*   The second stage starts completely fresh with `FROM scratch` — an empty image with nothing in it, not even an operating system.
    
*   `COPY --from=build /app /app` grabs **only** the compiled binary from the first stage and puts it into the clean second stage.
    
*   Everything else from the build stage (Ubuntu, Go compiler, source code) is thrown away and never shows up in your final image.
    

The result: the final image only contains your app's binary — nothing else. That's how the size drops so dramatically.

### 4\. "Scratch" vs "Distroless" — What's the Difference?

You'll notice the Dockerfile above uses `FROM scratch`. It's worth knowing there are actually two popular ways to build a minimal final image, and they're not quite the same thing:

*   `scratch` — a completely empty image. Zero files, zero OS, zero shell. It's the smallest possible starting point. Great for statically compiled languages like Go, where the binary doesn't need anything else to run.
    
*   **Distroless images** (like Google's `gcr.io/distroless` images) — almost empty, but they include just enough to run your app: things like basic runtime libraries and certificate files, but no shell, no package manager, and no unnecessary OS tools.
    

Both are much smaller and safer than a full OS image like `ubuntu` or `debian`. The choice between them usually comes down to this:

*   Use `scratch` when your app is fully self-contained and needs nothing else (common with Go).
    
*   Use **distroless** when your app needs a few basic runtime pieces (common with Java, Node.js, or Python apps), but you still want to avoid a full OS.
    

Either way, the goal is the same: ship only what your app truly needs to run, and nothing more.

### 5\. The Result: Image Size Comparison

Here's the size difference in practice. Add your two screenshots below when you paste this into Hashnode (drag and drop them into the editor where you see these markers):

**Before (single-stage build with full Ubuntu):**

![](https://cdn.hashnode.com/uploads/covers/67cf7a81a70907d5d60b29c7/32d18934-753e-4dca-909b-696165de0838.png align="center")

**After (multistage build with a clean final image):**

![](https://cdn.hashnode.com/uploads/covers/67cf7a81a70907d5d60b29c7/71b7e5bd-297a-49fe-ba99-6e448a7daee8.png align="center")

The difference is dramatic — going from a heavy, full-OS image down to just a few megabytes, since only the compiled binary remains in the final image.

### 6\. Why This Actually Matters

Smaller images aren't just about saving disk space. They also give you:

*   **Faster deployments** — smaller images push and pull faster, especially useful in CI/CD pipelines.
    
*   **Smaller attack surface** — no shell, no package manager, no extra OS tools means fewer places for an attacker to exploit.
    
*   **Lower costs** — less storage and bandwidth used, especially at scale with many containers.
    
*   **Cleaner, more predictable containers** — you know exactly what's running, because nothing extra snuck in.
    

### 7\. Try It Yourself

1.  Write a normal single-stage Dockerfile for any small app and build it. Note the image size using `docker images`.
    
2.  Rewrite it as a multistage build: name your first stage (`AS build`), then add a second stage starting from `scratch` or a distroless base image.
    
3.  Use `COPY --from=build` to bring over only what your app needs to run.
    
4.  Build again and compare the size using `docker images`. You should see a big drop.
    

### Key Takeaways

*   A normal Dockerfile ships your build tools along with your app — even though you only need the tools while building, not while running.
    
*   **Multistage builds** let you separate the "build" environment from the "run" environment, so only the final app makes it into your image.
    
*   `scratch` is a totally empty base image; **distroless** images are almost empty but include the bare minimum needed to run your app.
    
*   Smaller images mean faster deployments, a smaller attack surface, and lower costs.
    

* * *

*This note is part of my Docker learning journey — practical, hands-on notes as I learn.*