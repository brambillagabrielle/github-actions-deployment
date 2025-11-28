# GitHub Actions Deployment Project

[![github-actions-deployment](https://github.com/brambillagabrielle/github-actions-deployment/actions/workflows/deploy.yml/badge.svg?branch=main&event=push)](https://github.com/brambillagabrielle/github-actions-deployment/actions/workflows/deploy.yml)

## About the Project

This project was proposed in the [DevOps Path from roadmap.sh](https://roadmap.sh/projects/github-actions-deployment-workflow), with the objective of deploying a *[basic static page](https://brambillagabrielle.github.io/github-actions-deployment/)* in GitHub Pages using a **GitHub Actions workflow**.

## Project Structure

```
.github/
├── workflows/
│   └── deploy.yml
index.html
```

## Pipeline Structure

### Trigger

Triggers are **events that causes a workflow to start running**, such as [pushing changes to the repository](https://docs.github.com/pt/actions/reference/workflows-and-actions/events-that-trigger-workflows#push), specifically in the `main` branch, like used in the following case:

```yaml
on:
  push:
    branches: ["main"]
```

### Permissions

To **enable the job to deploy to GitHub Actions**, the default permissions for the token `GITHUB_TOKEN` are modified:

```yaml
permissions:
  contents: read
  pages: write
  id-token: write
```

### Build

The **runner** for both jobs (Build and later Deploy) is defined is [Ubuntu, in the latest version](https://docs.github.com/pt/actions/reference/runners/github-hosted-runners#standard-github-hosted-runners-for-public-repositories), provided by GitHub:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
```

The first step in the Build job is to execute a **Checkout**, which will **clone the related repository**:

```yaml
    steps:
      - name: Checkout
        uses: actions/checkout@v4
```

Next, since this pipeline is a **custom workflow to deploy the static site to GitHub Pages**, the action [`configure-pages`](https://docs.github.com/pt/pages/getting-started-with-github-pages/using-custom-workflows-with-github-pages#configuring-the-configure-pages-action) is used:

```yaml
      - name: Configure Pages
        uses: actions/configure-pages@v4
```

Last, the [`upload-pages-artifact`](https://docs.github.com/pt/pages/getting-started-with-github-pages/using-custom-workflows-with-github-pages#configuring-the-upload-pages-artifact-action) action is executed to package/compress all the files in the root directory (current, or `.`) for later deployment:

```yaml
      - name: Upload Pages Artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: .
```

### Deploy

After the Build job, the **Deploy is executed**, in which is defined the [`environment` as `github-pages`](https://docs.github.com/pt/pages/getting-started-with-github-pages/using-custom-workflows-with-github-pages#implantar-artefatos-do-github-pages) and the artifact generated in the last step is, finally, deployed:

```yaml
    deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```