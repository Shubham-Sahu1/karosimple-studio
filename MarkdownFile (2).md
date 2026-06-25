**KARO SIMPLE**

Sanity CMS Integration

| V1 — Studio Setup & Schema DefinitionHandoff Document for Antigravity |
| --- |
| Field | Detail |
| --- | --- |
| Project | karosimple.in — Marketing Website |
| Studio URL | cms.karosimple.in |
| Blog Route | karosimple.in/blog |
| Category Route | karosimple.in/blog/categories/[slug] |
| Document Version | V1 — CMS Setup Only (V2 covers Next.js integration) |
| Prepared By | Shubham — Co-founder, Karo Simple |
| Stack | Sanity v3, JavaScript, Vercel |

**SCOPE OF THIS DOCUMENT**

**NOTE** This is V1 only. It covers Sanity project creation, schema definitions, Studio deployment, and user access setup. Connecting the CMS to karosimple.in (Next.js) is covered in V2.

# 1\. Architecture Overview

Before any setup begins, understand what is being built and why each decision was made.

## 1.1 Content Flow

| Step | What Happens |
| --- | --- |
| 1. Write | Shubham or Chandan logs into cms.karosimple.in (Sanity Studio) |
| 2. Publish | Writer hits Publish — content saves to Sanity's hosted database |
| 3. Deliver | karosimple.in fetches posts via Sanity's CDN API (covered in V2) |
| 4. Display | Blog renders at /blog and /blog/categories/[slug] (covered in V2) |

## 1.2 Why Sanity — Not the Dashboard Backend

This question was evaluated before choosing Sanity. The reasons for not using the dashboard backend (app.karosimple.in) are:

*   Dashboard backend couples marketing availability to product uptime — if app goes down, blog 404s
*   Dashboard was not designed as a public content API — CORS, auth, and rate limiting would need to be added
*   Blog management and product management are different concerns and should stay separate
*   Sanity's CDN caches content at edge — Next.js ISR pages load fast, which improves Core Web Vitals and SEO
*   Chandan can publish without touching Git or the codebase

## 1.3 Deployment Targets

| What | Where |
| --- | --- |
| Sanity Studio (admin UI) | cms.karosimple.in — standalone Vercel project |
| Content database | Sanity Cloud — managed, no self-hosting needed |
| Marketing site | karosimple.in — existing Vercel project (untouched in V1) |
| PHASE 1 Sanity Project CreationCreate the Sanity account, project, and local Studio scaffold |
| --- |

## Step 1.1 — Create Sanity Account

1.  Go to sanity.io and sign up using the team email (or Shubham's Google account)
2.  Verify the email and complete account setup
3.  You will land on the Sanity dashboard at sanity.io/manage

**NOTE** Use one Sanity account for the entire project. Do not create separate accounts for each team member — add them as collaborators instead (covered in Phase 4).

## Step 1.2 — Create a New Sanity Project

1.  From sanity.io/manage, click Create new project
2.  Project name: Karo Simple Blog
3.  Dataset: select production (this is the default and correct choice)
4.  Plan: Free tier is sufficient for launch — upgrade if content volume grows
5.  Note the Project ID shown on the project overview page — you will need it in V2

| Setting | Value to Use |
| --- | --- |
| Project Name | Karo Simple Blog |
| Dataset | production |
| Plan | Free (upgrade later if needed) |
| API Version | 2024-01-01 (use this exact string in all API calls) |

## Step 1.3 — Scaffold the Studio Locally

Run the following commands on your local machine. Node.js 18+ must be installed.

mkdir karosimple-studio

cd karosimple-studio

npm create sanity@latest

When the CLI prompts you:

*   Log in or use existing account: select the account created in Step 1.1
*   Select project: choose Karo Simple Blog
*   Use default dataset configuration: Yes
*   Project output path: press Enter (use current folder)
*   Select project template: Blog (schema) — this gives you a base to modify
*   Add TypeScript support: No (project uses JavaScript)
*   Package manager: npm

**NOTE** The CLI creates a complete Studio project. Do not run sanity start yet — complete the schema work in Phase 2 first.

## Step 1.4 — Verify Local Studio Starts

npm run dev

Studio opens at http://localhost:3333. Confirm it loads. You can close it after confirming — there is more schema work to do before deploying.

| PHASE 2 Schema DefinitionDefine all content types that the blog will use |
| --- |

Schemas are defined in the schemaTypes/ folder inside the Studio project. Each file defines one document type. The CLI template creates some starter files — replace them with the schemas below.

**NOTE** All schema files are written in JavaScript (.js), not TypeScript. Do not add .ts extensions.

## 2.1 File Structure

The final schemaTypes/ folder should contain exactly these files:

schemaTypes/

index.js // registers all schemas

post.js // blog post document

category.js // category document

author.js // author document

blockContent.js // portable text definition

## 2.2 Schema: blockContent.js

This defines the rich text editor used in post body. Create schemaTypes/blockContent.js with the following content:

export default {

name: 'blockContent',

title: 'Block Content',

type: 'array',

of: \[

{

type: 'block',

styles: \[

{ title: 'Normal', value: 'normal' },

{ title: 'H2', value: 'h2' },

{ title: 'H3', value: 'h3' },

{ title: 'Quote', value: 'blockquote' },

\],

lists: \[{ title: 'Bullet', value: 'bullet' }, { title: 'Numbered', value: 'number' }\],

marks: {

decorators: \[{ title: 'Bold', value: 'strong' }, { title: 'Italic', value: 'em' }\],

annotations: \[

{ name: 'link', type: 'object', title: 'URL',

fields: \[{ name: 'href', type: 'url', title: 'URL' }\] },

\],

},

},

{ type: 'image', options: { hotspot: true } },

\],

}

## 2.3 Schema: author.js

Create schemaTypes/author.js:

export default {

name: 'author',

title: 'Author',

type: 'document',

fields: \[

{ name: 'name', title: 'Name', type: 'string', validation: Rule => Rule.required() },

{ name: 'slug', title: 'Slug', type: 'slug', options: { source: 'name' } },

{ name: 'role', title: 'Role', type: 'string' },

{ name: 'photo', title: 'Photo', type: 'image', options: { hotspot: true } },

{ name: 'bio', title: 'Bio', type: 'text', rows: 3 },

\],

}

## 2.4 Schema: category.js

Create schemaTypes/category.js:

export default {

name: 'category',

title: 'Category',

type: 'document',

fields: \[

{ name: 'title', title: 'Title', type: 'string', validation: Rule => Rule.required() },

{ name: 'slug', title: 'Slug', type: 'slug',

options: { source: 'title', maxLength: 96 },

validation: Rule => Rule.required() },

{ name: 'description', title: 'Description', type: 'text', rows: 2 },

\],

}

## 2.5 Schema: post.js

This is the most important schema. Create schemaTypes/post.js:

export default {

name: 'post',

title: 'Blog Post',

type: 'document',

fields: \[

{ name: 'title', title: 'Title', type: 'string', validation: Rule => Rule.required() },

{ name: 'slug', title: 'Slug', type: 'slug',

options: { source: 'title', maxLength: 96 },

validation: Rule => Rule.required() },

{ name: 'author', title: 'Author', type: 'reference', to: \[{ type: 'author' }\] },

{ name: 'coverImage', title: 'Cover Image', type: 'image',

options: { hotspot: true } },

{ name: 'categories', title: 'Categories', type: 'array',

of: \[{ type: 'reference', to: \[{ type: 'category' }\] }\] },

{ name: 'publishedAt', title: 'Published At', type: 'datetime',

validation: Rule => Rule.required() },

{ name: 'excerpt', title: 'Excerpt', type: 'text', rows: 3,

description: 'Short summary shown on blog listing page and in search results' },

{ name: 'body', title: 'Body', type: 'blockContent' },

{

name: 'seo', title: 'SEO', type: 'object',

fields: \[

{ name: 'metaTitle', title: 'Meta Title', type: 'string',

description: 'Keep under 60 characters' },

{ name: 'metaDescription', title: 'Meta Description', type: 'text', rows: 2,

description: 'Keep under 160 characters' },

\],

},

\],

preview: {

select: { title: 'title', author: 'author.name', media: 'coverImage' },

prepare({ title, author, media }) {

return { title, subtitle: author ? \`by ${author}\` : 'No author', media }

}

},

}

## 2.6 Schema: index.js

Update schemaTypes/index.js to register all schemas:

import blockContent from './blockContent'

import author from './author'

import category from './category'

import post from './post'

export const schemaTypes = \[blockContent, author, category, post\]

## 2.7 Full Schema Field Reference

Post schema — all fields at a glance:

| Field Name | Type | Required | Notes |
| --- | --- | --- | --- |
| title | string | Yes | Main headline of the post |
| slug | slug | Yes | URL-safe identifier, auto-generated from title |
| author | reference | No | Links to an Author document |
| coverImage | image | No | Cover photo with hotspot cropping enabled |
| categories | array[ref] | No | One or more Category references |
| publishedAt | datetime | Yes | Controls publish order on listing page |
| excerpt | text | No | Short summary for cards and meta description fallback |
| body | blockContent | No | Full rich text body of the post |
| seo.metaTitle | string | No | Overrides title in browser tab and Google |
| seo.metaDescription | text | No | Overrides excerpt in search result snippet |

Category schema — all fields:

| Field Name | Type | Required | Notes |
| --- | --- | --- | --- |
| title | string | Yes | Display name e.g. Review Tips, Restaurant Owners |
| slug | slug | Yes | Used in URL: /blog/categories/[slug] |
| description | text | No | Short blurb shown on category listing page |

Author schema — all fields:

| Field Name | Type | Required | Notes |
| --- | --- | --- | --- |
| name | string | Yes | Full name of the author |
| slug | slug | No | Auto-generated, used if author profile pages are added later |
| role | string | No | e.g. Co-founder, Digital Marketing Lead |
| photo | image | No | Author headshot with hotspot cropping |
| bio | text | No | Short bio shown on post detail page |
| PHASE 3 Studio Deployment to cms.karosimple.inDeploy Sanity Studio as a standalone Vercel project |
| --- |

## Step 3.1 — Push Studio to GitHub

1.  Initialize a Git repository inside the karosimple-studio folder
2.  Create a new private GitHub repo named karosimple-studio
3.  Push the local project to GitHub

git init

git add .

git commit -m "Initial Sanity Studio setup with schemas"

git remote add origin https://github.com/YOUR\_ORG/karosimple-studio.git

git branch -M main

git push -u origin main

## Step 3.2 — Create Vercel Project for Studio

1.  Go to vercel.com and log in
2.  Click Add New Project
3.  Import the karosimple-studio GitHub repository
4.  Framework preset: Other (Sanity Studio is not a Next.js project)
5.  Build command: npm run build
6.  Output directory: dist
7.  Click Deploy

**NOTE** Do not connect this Vercel project to the same project as karosimple.in. They must be two separate Vercel projects.

## Step 3.3 — Add Custom Domain cms.karosimple.in

1.  In Vercel, open the Studio project settings
2.  Go to Domains
3.  Add domain: cms.karosimple.in
4.  In your DNS provider (wherever karosimple.in DNS is managed), add a CNAME record:

| DNS Field | Value |
| --- | --- |
| Type | CNAME |
| Name / Host | cms |
| Value / Target | cname.vercel-dns.com |
| TTL | Auto or 3600 |

1.  Wait for DNS propagation (5 minutes to 1 hour)
2.  Vercel will auto-provision an SSL certificate — no manual action needed

## Step 3.4 — Configure CORS in Sanity

Sanity must be told which domains are allowed to make requests to its API.

1.  Go to sanity.io/manage and open the Karo Simple Blog project
2.  Navigate to API > CORS origins
3.  Click Add CORS origin and add each of the following:

| Origin URL | Purpose |
| --- | --- |
| http://localhost:3000 | Local Next.js dev server |
| http://localhost:3333 | Local Studio dev server |
| https://karosimple.in | Production marketing site |
| https://www.karosimple.in | www variant |
| https://cms.karosimple.in | Production Studio |

**NOTE** Do not enable Allow credentials unless you are building authenticated content. For a public blog, credentials are not required.

## Step 3.5 — Create an API Token for V2

This is done now and stored securely for when V2 begins.

1.  In sanity.io/manage, open the project and go to API > Tokens
2.  Click Add API Token
3.  Label: karosimple-marketing-site
4.  Permissions: Viewer (read-only is sufficient for public content fetching)
5.  Click Save and copy the token immediately — it will not be shown again
6.  Store it in a password manager or the team's secure notes

**NOTE** This token is only needed in V2 when the Next.js marketing site connects to Sanity. Do not commit it to Git or add it to any file in the Studio project.

| PHASE 4 User Access & RolesAdd team members and configure who can publish content |
| --- |

## Step 4.1 — Invite Team Members

1.  Go to sanity.io/manage and open the Karo Simple Blog project
2.  Click Members in the left sidebar
3.  Click Invite members
4.  Add each team member using the table below:

| Team Member | Role to Assign |
| --- | --- |
| Shubham (co-founder) | Administrator |
| Chandan (co-founder) | Editor |
| Antigravity developer | Administrator (during setup only, downgrade after launch) |

## Step 4.2 — Role Permissions Reference

| Role | What They Can Do |
| --- | --- |
| Administrator | Create/edit/delete all documents, manage schema, manage members, deploy Studio |
| Editor | Create, edit, publish, and delete content — cannot change schema or manage members |
| Viewer | Read-only — cannot publish or edit |

**NOTE** Chandan should be an Editor, not an Administrator. This prevents accidental schema changes from the Studio UI. Schema changes are only made by editing the code in the GitHub repository.

## Step 4.3 — First Content Entry (Seed Data)

After roles are set up, add the following as the first entries to confirm the Studio is working correctly:

*   Create two Author documents: Shubham and Chandan — add names, roles, and photos
*   Create three Category documents: Review Tips, Restaurant Owners, Local Business Growth
*   Create one draft Post using the post schema — do not publish it yet
*   Confirm the post preview in Studio shows title, author name, and cover image thumbnail

**NOTE** This seed data serves as the functional test for the schema. If any field throws an error on save, the schema definition has an issue that must be fixed before proceeding.

| PHASE 5 Verification ChecklistConfirm everything is working before handing off to V2 |
| --- |

Go through each item below before declaring V1 complete. Every item must be confirmed by a team member, not assumed.

| Checkpoint | How to Verify |
| --- | --- |
| Studio loads at cms.karosimple.in | Open the URL in a browser — no errors, login prompt appears |
| SSL certificate is active | Padlock icon appears in browser — no security warning |
| All four schema types appear in Studio sidebar | Post, Category, Author, Block Content are listed |
| Author document can be created and saved | Create Shubham as an author — no validation errors |
| Category document can be created and saved | Create Review Tips category with slug — no errors |
| Post document can be created with all fields | Create a draft post with title, author, category, body, SEO fields |
| Slug auto-generates from title | Type a post title — slug field auto-populates |
| Cover image upload works | Upload a test image — thumbnail appears in Studio |
| Rich text body accepts headings and links | In post body, add H2 heading and a hyperlink |
| Chandan can log in and create a post | Chandan logs in, creates a draft — confirm Editor role works |
| Project ID is documented | Note it from sanity.io/manage project overview |
| API token is stored securely | Token from Step 3.5 is in password manager |
| CORS origins are configured | All five origins are listed in sanity.io/manage > API > CORS |

# 6\. Handoff Notes for Antigravity

## What is out of scope for V1

*   No changes to karosimple.in (the Next.js marketing site) in this phase
*   No GROQ queries, no next/sanity package, no API route for revalidation
*   No blog pages rendered on karosimple.in — all of that is V2
*   No Razorpay or dashboard backend work is needed for this phase

## Repository Naming Convention

| Project | Suggested Repo Name |
| --- | --- |
| Sanity Studio | karosimple-studio |
| Marketing site (existing) | karosimple-marketing (or existing name) |

## Environment Variables — Studio Project

The Studio project requires no .env file for V1. The project ID and dataset are embedded in sanity.config.js by the CLI during scaffolding. Confirm they are correct:

// sanity.config.js — verify these values

projectId: 'YOUR\_PROJECT\_ID', // from sanity.io/manage

dataset: 'production',

**NOTE** The API token created in Step 3.5 is NOT added to the Studio project. It will be used only in the Next.js marketing project in V2.

## Schema Change Process

Once V1 is live, schema changes must follow this process to avoid breaking the Studio:

1.  Edit the relevant .js file in schemaTypes/
2.  Test locally by running npm run dev and confirming the Studio updates
3.  Commit to GitHub — Vercel auto-redeploys the Studio
4.  Verify the change is live at cms.karosimple.in

**NOTE** Never try to change schema directly in the Sanity dashboard UI. Schema is code — it lives in the repository. Any direct UI changes will be overwritten on the next deployment.

# 7\. What V2 Will Cover

V2 is a separate document and will be prepared after V1 is verified and signed off. It covers:

*   Installing @sanity/client and @portabletext/react in the marketing Next.js project
*   Creating src/marketing/lib/sanity.ts with the client configuration
*   Writing GROQ queries for post listing and single post pages
*   Building /blog/page.tsx — the blog listing page
*   Building /blog/\[slug\]/page.tsx — the single post page with generateMetadata for SEO
*   Building /blog/categories/\[slug\]/page.tsx — filtered post listing by category
*   Adding generateStaticParams to pre-render all blog and category pages at build time
*   Adding a Vercel webhook for instant revalidation when Chandan publishes a post
*   Adding Open Graph image support per post

**NOTE** V2 begins only after V1 checklist (Phase 5) is fully confirmed. Do not start integrating the Next.js site until the Studio is live and seed data is verified.

_End of V1 Document — Karo Simple Sanity CMS Setup_