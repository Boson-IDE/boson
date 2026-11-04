# Privacy Policy

_Last updated: 4 November 2026_

Boson IDE ("Boson") is a code editor and workspace tool that runs on your own computer. This policy explains what information Boson handles. The short version: **Boson has no accounts, no analytics and no telemetry, and the project does not collect your code or personal data.**

## What stays on your computer

Your files, settings, workspace history, trusted-folder list, window position and any tokens you enter are stored on your device only (in the app's user-data folder and the browser storage of the Boson window). Boson does not upload them to the project or to any server operated by the project.

## Network requests Boson makes

Boson contacts the internet only for features you use or that are described here:

| Feature | Who is contacted | What is sent |
| --- | --- | --- |
| Update check | GitHub (`api.github.com`) | A request for the project's latest release. Like any web request, GitHub sees your IP address. No identifier is added by Boson. |
| Package finder | Public package registries you search (npm, crates.io, PyPI, Go, NuGet, Packagist, RubyGems, pub.dev, Maven Central and others) | The search text or package name you enter. |
| Browser preview | The websites you open | Ordinary web requests for the pages you choose. Text that is not a web address is sent to Google Search. |
| Workspace Runtime | A Runtime you connect to (by default one on your own computer) | Your workspace files and commands, as needed for the features you use. A remote Runtime is operated by whoever runs it. |
| Git operations | The Git remotes you configure | What Git itself sends when you fetch, pull or push. |

Those services have their own privacy policies; Boson does not control them.

## Discord Rich Presence (optional, off by default)

If you switch on Rich Presence, Boson talks to the Discord app **running on your own computer** over a local connection. Through your Discord client it shares that you are using Boson, and, only if you leave the options on, your workspace name, the name of the file you are editing and its language. You can turn off each item separately or the whole feature in **Settings → Workbench → Discord Rich Presence**. Once Discord receives this, it is handled under [Discord's Privacy Policy](https://discord.com/privacy). Boson never sends it to the project.

## Data the project receives

None. The project does not operate a server that Boson reports to. If you open an issue through **Help → Send Feedback**, the information you put on that GitHub page (including the version and platform that Boson pre-fills) is public on GitHub and handled under GitHub's policies. You can edit or remove the pre-filled text before you submit.

## Children

Boson is a general developer tool and is not directed at children.

## Changes

If this policy changes, the new version will be published here with a new date.

## Contact

Questions about this policy: open an issue at [github.com/Boson-IDE/boson/issues](https://github.com/Boson-IDE/boson/issues).
