The site is deployed using Github Actions with Github Pages.

## Settings

In `Settings` -> `Pages` -> `Build and deployment` -> `Source`: Github Actions.

## New Post

```bash
hugo new content posts/java/my-first-post.md
```

Push the change to GitHub which will trigger the workflow defined in `.github/gh-pages.yml` to deploy the update using GitHub Actions.

## References

- [Hugo: Quick start](https://gohugo.io/getting-started/quick-start/)
- [Hugo: Host on GitHub Pages](https://gohugo.io/hosting-and-deployment/hosting-on-github/)

