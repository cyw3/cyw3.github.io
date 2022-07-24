# YalesonChan

## build
```bash
# init
npm install -g hexo-cli
# npm install hexo -g
# npm update hexo -g
hexo init

# test
hexo new test_my_site
hexo g
# hexo server
hexo s
# localhost:4000

# theme setting
npm install -S hexo-theme-icarus hexo-renderer-inferno
hexo config theme icarus

# deploy git
hexo g -d
```
