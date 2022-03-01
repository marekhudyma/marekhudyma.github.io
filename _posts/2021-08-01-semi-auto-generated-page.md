---
layout: post
title: "Semi auto generated page"
featured: false
author: marek
categories: java, startup 
type: post
image: '/assets/2021-08-01-semi-auto-generated-page/released-info.png'
comments: false
---

<figure>
  <img src="/assets/2021-08-01-semi-auto-generated-page/page-view.png" alt="Page view" />
</figure>


Some time ago I wanted to realize my tiny project: [Released.info](http://released.info) webpage, 
where I wanted to track if a given technology is already released. 

My business needs were:
* The hosting cost should be as small as possible - ideally `free`,
* I wrote a Java code that checks if a given technology is actually released (e.g. by searching some text on the webpage), 
* I wanted the page to be updated every day, so I can update a counter on the page. That why I call it `semi auto generated`.

# Chosen architecture 
I decided to host park my domain on Namecheap, without any good reason. 

My webpage is hosted on GitHub Pages, because it is a free hosting for repositories smaller than 1 GB. 
Additionally, GitHub Actions provide `2_000 minutes of free code execution` per month. 
I organized my code into two repositories: 
* First with my Java code, that is doing some actions:
    * executes once per day, or after merge operation, 
    * crawl pages and determinate if technology was released; if yes write an email, so I can validate it manually,
    * generate the webpage code (including updated counters),
    * push static webpage to second repository,
* serve static webpage.

# Github configuration

The GitHub configuration is placed in the repository `.github/workflows/maven.yaml`.
It executes by `cron`, once per 24 hours. It executes `mvn clean install` build and then pushes `output` directory to repository with static contewnt. 
Push is done by plugin: `dmnemec/copy_file_to_another_repo_action@main`.

```yaml
name: Java CI with Maven

on:
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]
  schedule:
    - cron: '0 */24 * * *'
jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - name: Set up JDK 17
      uses: actions/setup-java@v2
      with:
        java-version: '17'
        distribution: 'adopt'
    - name: Build with Maven
      run: mvn clean install
      
    - name: Push-Main
      uses: dmnemec/copy_file_to_another_repo_action@main
      env:
        API_TOKEN_GITHUB: ${{ secrets.API_TOKEN_RELEASED_INFO_GITHUB }}
      with:
        source_file: 'output/main/.'
        destination_repo: 'released-info/released-info-main'
        destination_folder: '/'
        user_email: 'contact@released.info'
        user_name: 'contactReleasedInfo'
        commit_message: 'Update html'
```

# Build 
To generate a static pages, I used an extremely easy method. Inside a test I put a code that generates a static webpage. 
Java code generates the webpage with templating system [Thymeleaf](https://www.thymeleaf.org/).
GitHub Pages right now offers 2_000 free minutes of execution per month. 
GitLab offers 400 free minutes, that should also fine. 

# Static webpage 
A static repository placed in GitHub Pages serves a static webpage. 
It uses [Content Delivery Network (CDN)](https://en.wikipedia.org/wiki/Content_delivery_network)
It offers `HTTPS` and it is very easy to point a domain to it. 

# Summary
In a described way I achieved a working solution that fullfil my needs: is free, but offers high quality service.
