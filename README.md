# Multipath TCP for Linux

This page is meant as a guide on the structure and particularities of this
website for those who want to contribute.

## The pages
- `index`, the landing page should contain basic context on this website and
  it's purpose as well as explanation on what is MPTCP. also a section dedicated
  to links directing to other MPTCP related content should be here.
- `setup`, is meant as a guide to users, helping them setup and use mptcp on
  their system.
- `implementation`, should contain any useful information on the use of MPTCP in
  a linux app.
- `debugging`, is meant as a source of solutions of how and what to use to debug
  MPTCP on a system.
- `details`, contains detailed explanation of the mechanisms of MPTCP.
- `questions`, the Q&A.
- `supported`, the list of currently supported apps.

## Front matter
At the top of all pages a section between `---`, contains information on the page.

All jekyll and just the docs values are supported and new ones have been added:
- `nav_titles` is a boolean. When set to true, the markdown titles of the page
  will be displayed in the navbar for easy access to a specific section.
- `titles_max_depth` is an integer that indicates the maximum level of titles
  included in the navbar. (*it directly correspond to the number of* `#`)

## Liquid markers
- `{: .ctsm}`, can be used on `<details>` tags to display "(click to see more)"
  in grey text after the `summary`

- `{: warining}`, `{: note}`, `{: info}` are callouts.
  Used before or after a paragraph, they will highlight it.

## Mermaid
mermaid graphs are supported in markdown files.


## Contributions
this website is open to contributions via pull request. A link in the bottom
left of pages point to the edition page on github.
