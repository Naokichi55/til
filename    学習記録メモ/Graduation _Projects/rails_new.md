Procfile.devを下記の状態からwebの行を削除
```
web: env RUBY_DEBUG_OPEN=true bin/rails server　←削除
js: yarn build --watch
css: bin/rails tailwindcss:watch

```
