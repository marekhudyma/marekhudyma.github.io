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



# Introduction 
In this article I will describe, how I realized my small site project using `free` and semi auto generated pages. 

# Problem description
Some time ago I wanted to realize my tiny project: [Released.info](http://released.info) webpage, 
where I wanted to track if a given technology is already released.

My functional requirements were:
* I wanted periodically check if a given technology has been released and regenerate page.
* I wanted the page to be updated every day, so I can update a counter on the page. That why I call it `semi auto generated`. Example of the counter is presented on the picture below. 

<figure>
  <img src="/assets/2021-08-01-semi-auto-generated-page/page-view.png" alt="Released info counter" />
</figure>

My non-functional requirements were:
* The hosting cost should be as small as possible - ideally `free`,

# Chosen architecture 

## Domain provider 
I decided to host park my domain on Namecheap, without any good reason. I just like the service. 

## Webpage hosting 
My webpage is hosted on GitHub Pages. It is a `free hosting` for repositories smaller than 1 GB. 
Additionally, GitHub Actions provide `2_000 minutes of free code execution` per month.
(GitLab offers 400 free minutes, that should also fine).

## GitHub repositories
I organized my code into two repositories: 
* First repository contains Java code, that is doing some actions periodically and after pull request merge:
    * crawl pages and determinate if technology was released; if yes send email to me, so I can validate it manually,
    * generate the webpage code (including updated counters),
    * push static webpage to second repository,
* Second repository simply serve static webpage.

## Manual configuration
The webpage is in the state, that I do not fully trust software decisions. Webpage crawling is pretty simple. I just search for some keywords with version. 
From the experience I realized that webpages and format of content are changing pretty often. I decided that on this stage I will notify myself when webpage changes. 
I do manual configuration change of the page, so it can be regenerated. It doesn't take much time. 
Maybe in the future the algorithm will be good enough to trust it. 

# Github configuration

The GitHub configuration is placed in the repository `.github/workflows/maven.yaml`.
It executes by `cron`, once per 24 hours. It executes `mvn clean install` build and then pushes `output` directory to repository with static content. 
Push is done by `GitHub plugin`: `dmnemec/copy_file_to_another_repo_action@main`.

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
Java code generates the webpage with [Thymeleaf](https://www.thymeleaf.org/) templating system.

# Static webpage 
A static repository placed in GitHub Pages serves a static webpage. 
It uses [Content Delivery Network (CDN)](https://en.wikipedia.org/wiki/Content_delivery_network)
It offers `HTTPS` and it is very easy to point a domain to it. 

# Summary
In a described way I achieved a working solution that fulfill my needs: is free, but offers high quality service.
