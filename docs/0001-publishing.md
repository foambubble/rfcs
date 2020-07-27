---
Start Date: 2020-07-27
RFC PR: (leave this empty)
Foam Issue: (leave this empty)
---

# 0001 Publishing

This RFC is more an umbrella covering the entire publishing story in order to discuss the architecture as opposed to implementation details.

## Summary

- Replace Jekyll publishing by one (or more) [Gatsby](https://gatsbyjs.org) themes (one for each use-case)
- Support an array of hosting solutions (GitHub pages, Netlify, etc.) by providing files to drop into a Foam workspace (`.github/workflows/publish.yml`, `.netlify.toml`, etc.)
- Provide a recipe to customize a theme
- Provide a recipe to use other Git providers

## Motivation

One of the goals of Foam is to make it easily to share your thoughts. That imply a way to publish a Foam workspace on the internet.

At the moment, that goal is fulfilled by leveraging the GitHub built-in publishing system: Jekyll + GitHub pages. While it works great for simple websites, we feel that we are going to rapidly outgrow it if we want to surface backlinks, block embeds, etc.. We also want to decouple the GitHub Foam template from the publishing part. That means that the code source of the website shouldn't be bundled but instead we only include a configuration file if needed.

Gatsby is a popular static website generator (SSG) built on React and GraphQL. It provides a way to share functionalities (a ["theme"](https://www.gatsbyjs.org/docs/themes/what-are-gatsby-themes/)) while retaining full control over the customization if the user wants to (see [Shadowing a theme](https://www.gatsbyjs.org/docs/themes/shadowing/)). Building a Gatsby project produces a `public` folder ready to be deployed on any static hosting provider, making it the perfect candidate for an extensible system with good defaults and a huge community to help. Gatsby themes can use any Npm packages, meaning we will be able to leverage the different packages made by foam-core to extract information needed from a Foam workspace.

GitHub pages is still a great hosting solution for a simple way to get things online. It will stay the default way of publishing a Foam workspace but we want to support other hosting solutions as first class citizens. Each of them have different pros and cons. The main advantage of GitHub pages is that it's by GitHub so you don't need another account to deploy something. The main disadvantage is that it is not available for free users on private repos. Luckily it wouldn't be too complicated to support multiple hosting solutions by using Gatsby.

## Detailed design

- A repo for each Gatsby theme (`gatsby-theme-foam-blog`, `gatsby-theme-foam-digital-garden`, `gatsby-theme-foam-team-doc`, etc.). Each theme should be discussed in follow-up RFCs.
- Each theme should try to have consistent options so that trying a theme is as simple as changing its name in the configuration file.
- A repo to host hosting configuration

## Drawbacks

## Alternatives

## Adoption strategy

## Unresolved questions

- Using a theme means adding a `package.json` and a `gatsby-config.js` files to a Foam workspace.
  - Shall it be included by default in the template?
  - If so with what theme?
- Using a hosting solution means copy pasting the configuration files in your repo
  - Could this be automated with a cli `npx foam setup-host netlify`?
  - Shall the deployment to GH pages be included by default in the template?
