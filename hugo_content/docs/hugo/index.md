---
title: 'Install Whisper Theme'
date: 2019-02-11T19:27:37+10:00
draft: false
weight: 5
summary: "For all the previous steps you must write documentation for each service. How to deploy, describe how it works with links to your GitLab repositories. Add whatever you think is necessary."
---

### Create a new Hugo site

```
$> hugo new site /mnt/hugo
```

This will create a fresh Hugo site in the folder `/mnt/hugo`.

### Install theme

Copy or git clone this theme into the sites themes folder `mynewsite/themes`

##### Install with Git

```
$> cd /mnt/hugo
$> git clone https://github.com/jugglerx/hugo-whisper-theme.git themes/hugo-whisper-theme
```

### Add the contents

The fastest way to get started is to copy the example content and modify the included `config.toml`

##### Copy contents

Copy the entire contents of the `exampleSite` folder to the root folder of your Hugo site.
```
$> cp -a themes/hugo-whisper-theme/exampleSite/. .
```
Then copy the docs files to the root folder
```
$> cp -a $git_folder/hugo_content/. content/
```
You can edit all the index.md inside contents directory to made all the changes you want

##### Update config.toml

After you copy the `config.toml` into the root folder of your Hugo site you will need to update the `baseURL`, `themesDir` and `theme` values in the `config.toml`

```
baseURL = "https://hugo.etna.student"
themesDir = "themes"
theme = "hugo-whisper-theme"
```

### Run Hugo

After installing the theme for the first time, regenerate the Hugo site.

```
$> kubectl rollout restart deploy/hugo -n clo4
```

Now enter [`hugo.student.etna`](https://hugo.etna.student) in the address bar of your browser.

