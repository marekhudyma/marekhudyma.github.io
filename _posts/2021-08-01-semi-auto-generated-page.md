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
In this article I will describe, how I realized my small site project using `free` services and semi auto generated pages. 

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
I decided to park my domain on Namecheap, without any good reason. I just like the service. 

## Webpage hosting 
My webpage is hosted on GitHub Pages. It is a `free hosting` for repositories smaller than 1 GB. 
Additionally, GitHub Actions provide `2_000 minutes of free code execution` per month.
(GitLab offers 400 free minutes, that also should also fine).

## GitHub repositories
I organized my code into two repositories: 
* First repository contains Java code, that is doing some actions periodically and after pull request merge:
    * crawl pages and determinate if technology was released; if yes send email to me, so I can validate it manually,
    * generate the webpage code (including updated counters),
    * push static webpage to second repository,
* Second repository simply serve static webpage.

You can follow the process in the diagram below
<figure>
  <img src="/assets/2021-08-01-semi-auto-generated-page/regeneration_html_flow.png" alt="Regeneration of HTML flow" />
</figure>

## Manual configuration
The big elephant in the room are manual intervention taken by human. The webpage is in the state, that I do not fully trust software decisions. Webpage crawling is implemented in pretty simple way. I just search for some keywords with version. 
From the experience I realized that webpages and format of content are changing pretty often. I decided that on this stage I will notify myself when webpage changes. 
I do manual configuration change of the page, so it can be regenerated. It doesn't take much time. 
Maybe in the future the algorithm will be good enough to trust it. 

# Generation of static webpage
To generate a static pages, I used an extremely easy method. Inside a test I put a code that generates a static webpage. 
Java code generates the webpage with [Thymeleaf](https://www.thymeleaf.org/) templating system.

I will show only small example of HTML page with templates:

```HTML
<span class="card-text">
  <th:block th:utext="${configuration.getMainTitle}" />
</span>
```

Later it is process by Java code:
```Java
private static String generateHTMLPage(List<Configuration> configurations) {
  TemplateEngine templateEngine = new TemplateEngine();
  ClassLoaderTemplateResolver resolver = new ClassLoaderTemplateResolver();
  resolver.setPrefix("/templates/main/");
  resolver.setSuffix(".html");
  resolver.setCharacterEncoding("UTF-8");
  resolver.setTemplateMode(TemplateMode.HTML);
  templateEngine.setTemplateResolver(resolver);
  Context ct = new Context();
  ct.setVariable("configurations", configurations);
  ct.setVariable("commentedTimestamp", configurations.get(0).getCommentedTimestamp());
  return templateEngine.process("index.html", ct);
}
```
The values for placeholders are passed here: `ct.setVariable("configurations", configurations);`.
Generated HTML file I save on disk to temporary directory. These files are later pushed by GitHub Actions plugin to separate repository. 

# Github Action configuration

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
## GitHub token
Your repository need to have right to push code (`HTML pages`) you need to somehow configure permissions to do it.
In the GitHub actions configuration, in the plugin: `dmnemec/copy_file_to_another_repo_action@main`, I configured it to use my token: `API_TOKEN_GITHUB: ${{ secrets.API_TOKEN_RELEASED_INFO_GITHUB }}`.
You can generate a new token here: [https://github.com/settings/tokens](https://github.com/settings/tokens)
and I configure it correctly. For me, it was selecting permissions: `repo` and `workflow`.

# Static webpage
A static repository placed in GitHub Pages serves a static webpage.
It uses [Content Delivery Network (CDN)](https://en.wikipedia.org/wiki/Content_delivery_network)
It offers `HTTPS` and it is very easy to point a domain to it.
By default, your webpage is available by url: `https://<user>.github.io`.

# Domain configuration 
I would like my page to be avaiable under the url: `https://released.info` and I would like to make some other funny subdomains `https://is.java.released.info`. 

## Namecheap configuration
GitHub instruct users to point `A records` to 4 GitHub DNS addresses.
<figure>
  <img src="/assets/2021-08-01-semi-auto-generated-page/namecheap_dns_configuration.png" alt="Namecheap dns configuration" />
</figure>

## GitHub pages configuration
After DNS Namecheap configuration, we can configure GitHub pages.
In the `settings` -> `pages` just: 
* point which branch serves the content, 
* what is the custom domain name, 
* click checkbox to enable `HTTPS`.

<figure>
  <img src="/assets/2021-08-01-semi-auto-generated-page/github_pages_custom_domain.png" alt="Namecheap dns configuration" />
</figure>

## Subdomains 
As I mention earlier, I would like to have many subdomains like `is.java.released.info`, `is.python.released.info`, etc. 
For every subdomain I needed to create another CNAME `record`.
<figure>
  <img src="/assets/2021-08-01-semi-auto-generated-page/namecheap_dns_configuration_subdomain.png" alt="Namecheap dns configuration for subdomain" />
</figure>

Additionally, I needed to create a separate GitHub repository for every domain.

# Email sending via Gmail
As I wrote earlier, I use email to notify myself about the changes in the system. I used it, because it was very easy to configure. 
To send emails programmatically, Gmail created an [instruction how to generate password for the program](https://support.google.com/accounts/answer/185833?hl=en).
Generally [in your account](https://myaccount.google.com) go to `security` -> `Sign in with Google` -> `App Passwords`.
Later use it as standard SMTP server. 

# Receiving emails. 
I decided that I would like my contact email look professionally: `contact@released.info`. I didn't want to spend much time to configure SMTP server. 
I just used a free `https://improvmx.com` service to redirect emails from `contact@released.info` to my personal email account. 
So from outside it looks professionally, whereas when I respond back, my personal email will be fully visible. Till now, I didn't need to respond to anybody ;-)
To use it you need to set `MX Record` in your domain provider. 

# Disadvantages
I am pretty satisfied how the solution works. Whereas, there are some disadvantages: 
* From time to time, some manual action is required. Webpages are changing constantly. I think I some small human intervention will be still needed from time to time. 
* Around once per quarter I have a problem that some subdomain stops to refresh the webpage. HTML is pushed to repository, but webpage is stale. I didn't use my free quote for minutes.. 
In this case I simply delete the repository with HTML and recreate it. Then it works again.

# Summary
In a described way I achieved a working solution that fulfill my needs: is free, but offers good enough service.