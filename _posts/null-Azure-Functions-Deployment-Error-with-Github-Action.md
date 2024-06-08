---
id: e38d7a7e-ac65-428e-a59d-c724e2cc61c8
title: Azure Functions Deployment Error with Github Action
created_time: 2024-04-30T03:00:00.000Z
last_edited_time: 2024-06-08T03:53:00.000Z
tags:
  - Github
status: Ready
layout: post

---

## Backgroud

I’m using Github Action to deploy my function to Azure, everything works smoothly until last time I commit.

I did two things.

*   Change Environment Variables in Azure Function. (not the step cause problem)

*   Update requirements.txt

Yes, you will not believe it, requirement.txt.

```bash
pip freeze > requirement.txt
```

I discover this problem when I diff files one by one.

```bash
git diff <commit-hash> -- <file-path>
```

And I found this line below.

```bash
diff --git a/requirements.txt b/requirements.txt
Binary files a/requirements.txt and b/requirements.txt differ
```

I don’t think it reasonable for a .txt type of file identify as binary.

For some reason, `requirements.txt` is encoded in `UTF-16 LE` .

## Resolve

```bash
git checkout <commit-hash> -- <file-path>
```
