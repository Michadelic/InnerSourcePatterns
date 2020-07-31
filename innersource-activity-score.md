## Title 
InnerSource Activity Score

## Patlet
The InnerSource activity score is a numeric value that represents the (GitHub) activity of an InnerSource project. It can be used to create a ranked list of projects for the [InnerSource portal](innersource-portal.md) so that potential contributors can easily find the most active projects. It is derived automatically from GitHub repository statistics and enriched with manual evaluations. 

## Problem

Potential contributors want to find active projects with a lot of traction, but also fairly new and enthusiastic projects with a need for new contributions - easy and fast.

**But in which order** shall InnerSource projects be presented? Typical ranking KPIs like *GitHub Stars*, *Number of Forks*, *Number of Commits*, *Lines of Code*,  *Last Update* aren't very accurate when it comes to activity of a project.

A new metric derived from several KPIs is needed to define a reliable and versatile score for a project's activity level.
It can be used to sort or rank projects according to their activity level.

## Story

When InnerSource is practiced for a long time or scales beyond a certain number of projects (let's say 50 to give a meaningful threshold) it is hard to find the currently most popular and active InnerSource projects. Projects that exist for a long are well-known but may no longer be very active. Fairly new projects on the other hand don't have a repuation or an active community yet.

A list of InnerSource projects should not be considered a static resource, but an exciting place to discover and explore new and active projects, just like a news page listing the most interesting topics of the day first. Thus it is beneficial when the order of the projects is regularly updated and changes according to the project's popularity and activity.   

These considerations let to a first prototype to calculate an InnerSource activity score, which worked surprisingly well and determines an ever-changing order of projects according to their activity.

## Context

Discovering InnerSource projects can be facilitated with the [InnerSource Portal](innersource-portal.md) and the [Gig Marketplace](gig-marketplace.md) pattern, or by promoting projects on other communication channels and platforms. The activity score defines a default order or ranking in which projects are presented to the community. 

## Forces

Automated KPIs that can be fetched by querying the GitHub API are only part of the truth. What about code quality, the availability of good documentation, or an active and helping community that makes the project a fun place to contribute.

Such "soft" KPIs would have to be manually or semi-automatically added to the calculation and the resulting score. For now, we focus on a fully automated solution. If tools exist that provide more context for the project, like a code coverage reporting, they can easily be worked in.

## Sketch (optional)

```//TODO: add sketch here ;-)```

## Solutions

The InnerSource activity score can be derived automatically from common parameters like GitHub stars, watches, and forks. In addition, it considers activity parameters like last update and creation date of the repo to give young projects with a lot of traction a boost.
Projects with contributing guidelines and issues (public backlog) may get a higher ranking.

All of this can be fetched and calculated automatically using the GitHub API (see [reference implementation](TODO) below). The code below assumes the variable `repo` contains an entity fetched from the GitHub API. Manual adjustments can be made on top if needed.

``` javascript
// calculate a virtual InnerSource score from stars, watches, commits, and issues
function calculateScore(repo) {
    // weighting:
    // forks and watches count most, then stars, add some little score for open issues, too
    let iScore = 1 + repo["forks_count"] * 5 + repo["watchers_count"] + repo["stargazers_count"] / 3 + repo["open_issues_count"] / 5;
    let iDaysSinceLastUpdate = (new Date().getTime() - new Date(repo.updated_at).getTime()) / 1000 / 86400;
    // updated at this month: adds a bonus between 0..1 to overall score (1 = updated today, 0 = updated more than 100 days ago)
    iScore = iScore * (1 + (100 - Math.min(iDaysSinceLastUpdate, 100)) / 100);
    // new repos (created in the last 365 days) that have been updated recently (in the last 30 days) get a boost of up to 1000
    let iDaysSinceCreation = (new Date().getTime() - new Date(repo.created_at).getTime()) / 1000 / 86400;
    let iBoost = (1000 - Math.min(iDaysSinceLastUpdate, 100) * 10);
    iScore += (iDaysSinceCreation < 365 ? (iDaysSinceLastUpdate < 30 ? iBoost : 0) : 0);
    // give projects with contribution guidelines (CONTRIBUTING.md) file a static boost of 100
    iScore += (repo["_InnerSourceMetadata"] && repo["_InnerSourceMetadata"]["guidelines"] ? 100 : 0);
    // final score is a rounded value starting from 0
    iScore = Math.round(iScore - 1);
    // add score to metadata on the fly
    repo._InnerSourceMetadata = repo._InnerSourceMetadata || {};
    repo._InnerSourceMetadata.score = iScore;
    return iScore;
}
```

## Resulting Context

Projects can be sorted and presented by the InnerSource activity score to give a meaningful order in a list or portal. The activity score can be calculated on the fly or in a background job that evaluates all projects on a regular basis and stores a list of results.

A crawler that regularly searches all InnerSource repositories (e.g. tagged with a certain label in an enterprise GitHub instance) can be a helpful solution as well. It provides a ranked list of projects that can be used as an input for tools like the [InnerSource Portal](innersource-portal.md), a search engine, or an interactive chat bot. 

## Rationale

The InnerSource activity score is a simple calculation based on the GitHub API. It can be fully automated and easily adapted to new requirements. 

## Known Instances (optional)
Used in SAP's InnerSource project portal to define the default order of the InnerSource projects. It was first created in July 2020 and is fine-tuned and updated frequently ever since.

When proposed to InnerSourceCommons in July 2020, this pattern emerged.

## Status (optional until merging)
First Draft: 30th August 2020  

## Author(s) (optional)
[Michael Graf (SAP)](mi.graf@sap.com)

## Acknowledgements (optional)

Thank you to the InnerSource Commons Community for lightning-fast advice and a lot of input to feed this pattern!
