# Swarm of Biz

A GitHub Pages website that displays business-card images from public Blogger posts at https://swarmofbiz.blogspot.com/.

## Publishing

1. Create a Blogger post with the business name as its title.
2. Insert the business-card image as the **first image** in the post.
3. Apply exactly one of the 15 service labels shown on the website (e.g. `Transportation`).
4. Publish. The site reads the public Blogger feed when visitors load it; no GitHub update is required.

## GitHub Pages

Upload `index.html`, `style.css`, and `script.js` to the root of your public `msafrain/swarmofbiz` repository. In Settings → Pages, choose **Deploy from a branch**, `main`, `/ (root)`.

## Notes

- Homepage fetches the six newest posts. Categories paginate through all matching posts (up to a generous safety limit).
- Cards link to their original Blogger post for now; custom card detail pages can be added later.
- The bee is a temporary emoji placeholder, not the final crying-bee illustration.
- Blogger public feed access and image delivery must be enabled. Some Blogger feed or template configurations may require adjustments.
- GitHub Pages usage terms should be checked before using it for a commercial directory.
