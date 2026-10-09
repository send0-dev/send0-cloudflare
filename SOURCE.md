# Corresponding source

This repository is built, not written. Its contents are generated from send0 0.2.0, commit:

https://github.com/send0-dev/send0-v2/tree/eab68c4eb05b09c07759e2d40d0321a65caacdb4

That commit is the complete corresponding source (GNU AGPL v3, section 6) of the prebuilt Worker in
`worker/` and the dashboard in `public/`. To rebuild this folder from it:

```sh
git clone https://github.com/send0-dev/send0-v2 && cd send0-v2 && git checkout eab68c4eb05b09c07759e2d40d0321a65caacdb4
pnpm install --frozen-lockfile
node apps/cloudflare/scripts/build-template.mjs ../send0-cloudflare --source-sha=eab68c4eb05b09c07759e2d40d0321a65caacdb4 --version=0.2.0
```

If you modify send0 and let others use it over a network, AGPL section 13 requires you to offer them
your modified source too.
