# Documentation Guidelines

## PRs for every change

All changes have to be done in a separate Branch. As soon as the changes are done please open a PR. A Github Action runs with every commit to a Branch and checks if the documentation can be build (including a broken links check).

## Structure

- Name Markdown files according to the title. This makes it easier to find a file. Example: **Introduction to Camunda Cloud** --> `introduction-to-camunda-cloud.md`. Use the file name as internal document id to reference in `sidebars.js`.
- Sub catagories have to be placed in their own sub directories. Example: Guides/Getting Started can be found in `docs/guides/getting-started`.

## Adding a new documentation page

1. Select the corresponding directory.
2. Add the document id to `sidebars.js`.

## Moving an existing page

1. Identify the page, pages, or directory and relocate it in the file structure.
2. Update `sidebars.js` to fit the new location.
3. Add a redirect/rewrite rule to the top of `.htaccess`.

## Review Process

After the proprosed change is finished open a GitHub PR and assign at least one reviewer, it is good to pick a reviewer who is expert in the matter of the change. If unsure about who to pick choose one of the corresponding team representives, and they will take care of delegating the issue:

- Cloud: @urbanisierung
- Zeebe: @pihme
- Operate/Tasklist: @ralfpuchert
- DevRel: @akeller
- Product Management: @felix-mueller
- Fallback: @menski

In case you don't know who to assign choose @menski and he will delegate.

As a reviewer feel free to merge any PR which you feel comfortable with after your review. If you have questions, concerns, or feel that you are not the right person to review the PR please make this transparent to the PR author so they can clarify this.
