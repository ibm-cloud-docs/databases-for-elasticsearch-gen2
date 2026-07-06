# Databases for Elasticsearch Gen 2 documentation

This repo contains documentation for the Databases for Elasticsearch Gen 2 service.

## About Gen 2

This repository contains documentation for Databases for Elasticsearch on the Gen 2 (VPC-based) platform. Gen 2 offers:
- Isolated Compute hosting model with dedicated resources
- Private endpoints only
- Enhanced security and performance
- VPC-based networking

For Gen 1 (Classic) documentation, see the .

## Contacts for this repo

- Andrea Lang: ANDREAL, andreal@de.ibm.com

## Creating content

Follow these steps to add to the deployment guide docs.

:information_source: **Tip:** If you'd rather give feedback about the documentation, create an .

### Before you begin
Set up your local development environment.

- Visual Studio Code is the recommended editor. For more information, see .
- You can install a Markdown linter in Visual Studio Code that works with IBM Cloud docs. For more information, see .

### Drafting content

All content starts from the `source` branch.

1.  Clone this repo if you have write privileges. Otherwise, fork the repo.
1.  Create a working branch from the `source` branch.
1.  Make your changes to the Markdown content.

    - Cloud docs uses Markdown with a few custom extensions to source the docs. For tips about how to structure and style your docs with IBM Cloud Markdown, see  in "Creating solution, deployment, and migration guides".
    - Cloud docs also supports controlling content with tagging. For example, content within the staging code tags is not displayed to the public. For more information, see .

1.  Commit your updates.
1.  Create a pull request from your branch or fork to `source`.

    1.  A Jenkins job runs that commits content to the `draft` and `review` branches and opens a pull request to the `publish` branch.
    1.  After a few minutes, you can see your changes in the IBM Cloud docs framework. Changes to `draft` are available at the internal `/docs-draft/` location . Changes to `review` are available at the pre-production `/docs/` location .

:information_source: **Tip:** Content that is tagged with draft, review, or staging, or feature tags are built and promoted only to the internal location and is not included in the pull request to the `publish` branch for production.

## Publishing to production

When content is committed to the `source` branch, a pull request is created from a branch called `next-prod-push` to the `publish` branch. So, changes can't go into the branch until they are reviewed, edited, and approved by a one of the doc maintainers.

The documentation team reviews your PR and might request changes.

- If the changes are relatively straightforward and self-contained, such as a corrected typographical error or a rewritten sentence, we will approve and merge them after issues are addressed.
- If the changes are more extensive, such as a significant rewrite or entirely new content, the documentation team might need to make revisions for editorial or style reasons.⁠ In this case, we might open a new PR against your branch with our proposed revisions.⁠ You can then review these revisions and incorporate the changes into your branch.⁠ After the documentation team is satisfied with the proposed changes, we merge your PR.⁠
When changes are merged to the `production` branch, they are built and published to production at https://cloud.ibm.com/docs/databases-for-elasticsearch-gen2.

### Guidelines for merging a PR

As someone with merge responsibilities, follow these guidelines and practices to make sure that changes are released consistently.


- Make sure that the merge commit message is clear and specific.

    :exclamation: **Important:** Do not expose IBM Confidential information in your commit message to `publish`. Commits made to the `publish` branch become public record. When you merge to the `publish` branch, the source is mirrored in a public GitHub repo at https://github.com/ibm-cloud-docs/databases-for-elasticsearch-gen2 so that customers can view and contribute to the source.
- Squash and merge

    - Use the **Squash and merge** option when you merge a PR. Status checks prevent the merge if the squash and merge method is not used. For more information, see [Squashing your merge commits](https://docs.github.com/repositories/configuring-branches-and-merges-in-your-repository/configuring-pull-request-merges/about-merge-methods-on-github#squashing-your-merge-commits).


## Monitoring

After a build is triggered by a commit or merge, you can monitor progress.

### Monitoring builds

The Slack channel [#docs-databases-for-elasticsearch](https://ibm.enterprise.slack.com/archives/C02D2K5JN10) displays information about builds.

### Monitoring content quality

You can monitor your content quality on the Content Quality Dashboard (CQD): . The CQD for docs content is updated daily For more information, see .

## More information

- Learn about the suggested content for each type of solution docs at .

## Output

The output from the `draft` branch is available on the staging server:

- 

The output from the `publish` branch is mirrored on [github.com](https://github.com/ibm-cloud-docs/databases-for-elasticsearch-gen2) and available in production:
- [Production Docs](https://cloud.ibm.com/docs/services/databases-for-elasticsearch-gen2)

## Resources

- 
- 

For general questions about IBM Cloud Databases, see the [icd-questions channel](https://ibm-cloudplatform.slack.com/messages/C534XRCF3/).

## Blog Submissions

To submit a blog, go . Ian Smalley handles everything blog-related, so any updates go through him. Use the same form to update, just make a note that it's an update, not a new post.
