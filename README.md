# Qiita-Sync Template

1. Click "Use this template" button, and create your GitHub repository.

2. Generate "Qiita Acccess Token" of at your Qiita site.

3. Open "Settings" >> "Secrets" >> "Actions" >> "New repository secret"

4. Input "QIITA_ACCESS_TOKEN" to "Name" and the Qiita Access Token to "Value", and click "Add secret"

5. "Actions" >> "Qiita Sync" >> "Run workflow" pulldown menu >> click "Run workflow" button

6. Your Qiita articles will be downloaded to your GitHub repository.

7. Rewrite this README for your own with the badge below (Replace \<Your-ID\> and \<Your-Repository\>)

```
![Qiita Sync](https://github.com/<Your-ID>/<Your-Repository>/actions/workflows/qiita_sync_check.yml/badge.svg)
```

Find more details in:

- English:  [Qiita-Sync README](https://github.com/ryokat3/qiita-sync)
- Japanese: [GitHub連携でQiita記事を素敵な執筆環境で！](https://qiita.com/ryokat3/items/d054b95f68810f70b136)
