# Corresponding source

This repository is built, not written. Its contents are generated from send0 0.2.1, commit:

https://github.com/send0-dev/send0-v2/tree/985e47b5776bbd47ef5a0ca4556e88dc89a18e57

That commit is the complete corresponding source (GNU AGPL v3, section 6) of the prebuilt Worker in
`worker/` and the dashboard in `public/`. To rebuild this folder from it:

```sh
git clone https://github.com/send0-dev/send0-v2 && cd send0-v2 && git checkout 985e47b5776bbd47ef5a0ca4556e88dc89a18e57
pnpm install --frozen-lockfile
node apps/cloudflare/scripts/build-template.mjs ../send0-cloudflare --source-sha=985e47b5776bbd47ef5a0ca4556e88dc89a18e57 --version=0.2.1
```

If you modify send0 and let others use it over a network, AGPL section 13 requires you to offer them
your modified source too.
