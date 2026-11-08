# PCC LCP Evaluators

A single web page that an LCP participant uses to build their evaluator list. They add names,
relationships and email addresses, and the page counts each group live and warns them when a
group is too small to be reported. They download a CSV and send it back.

Live at: **https://ohikal26.github.io/PCC-LCP-Evaluators/**

---

## Why a web page and not a spreadsheet

The thing that goes wrong in LCP setup is not data entry. It is a participant naming two peers
and losing the whole peer view from their report, which nobody notices until the report is built.
This page counts each group as they type and tells them while they can still do something about
it.

It also means no file version problems, no Excel on a locked-down corporate machine, and Arabic
names that survive the round trip.

---

## What the participant does

1. Opens the link.
2. Types their own name and email at the top.
3. Adds evaluators, or pastes a list from a spreadsheet or Outlook.
4. Watches the five counters until none of them are flagged.
5. Presses **Download the list** and emails the file back.

If downloads are blocked on their machine, **Copy instead** puts the same text on the clipboard
to paste into an email.

The file is named `LCP_Evaluators_[Their_Name]_[date].csv`, so you can tell whose list it is
without opening it.

---

## What comes out

A CSV with a header row and one line per person, sorted into group order:

```
Name,Relationship,Email
Youssef Kamal,Boss,youssef@example.com
Laila Abdel Aziz,Peer,laila@example.com
"Smith, John",Other,john@example.com
رشا محمود,Direct Report,rasha@example.com
```

Names containing commas are quoted. The file carries a byte order mark so Excel opens Arabic
correctly instead of turning it into symbols. Line endings are CRLF, which is what Excel expects.

If the LCP importer wants different column headings, change the one line in `index.html` that
reads `var lines = ["Name,Relationship,Email"];`.

---

## The rules it enforces

These are set in one block at the top of the script in `index.html`:

```js
var MIN_TOTAL = 10;
var GROUPS = [
  {key:"Boss",          label:"Boss",           min:1, solo:true,  opt:false},
  {key:"Boss's Boss",   label:"Boss's Boss",    min:1, solo:true,  opt:true },
  {key:"Peer",          label:"Peers",          min:3, solo:false, opt:false},
  {key:"Direct Report", label:"Direct Reports", min:3, solo:false, opt:false},
  {key:"Other",         label:"Other",          min:3, solo:false, opt:true }
];
```

- `min` is how many that group needs before it is reported on its own.
- `solo: true` means the group is shown even with one person in it, so it never gets the
  "needs three" warning. Boss and Boss's Boss are set that way.
- `opt: true` means leaving the group empty is fine and draws no warning. A group marked `opt`
  that has one or two people in it still warns, because a part-filled group is the problem.

**Check these against the current LCP rules before you send the link out.** The three-per-group
minimum and showing the boss alone are the convention as I understand it, not something I
verified against Leadership Circle's current documentation. Everything is in that one block, so
correcting it is a two-minute edit and nothing else in the page needs touching.

---

## How to publish it

Same route as the surveys and the close-out page.

1. On github.com, click **+** then **New repository**.
2. Name it exactly `PCC-LCP-Evaluators`. Set it **Public**. Do not tick "Add a README file".
   Click **Create repository**.
3. Click **uploading an existing file**.
4. Drag in `index.html`, `README.md` and `.nojekyll`. Click **Commit changes**.
5. **Settings** → **Pages** → Source **Deploy from a branch**, branch **main**, folder
   **/ (root)**. Save.
6. Give it a couple of minutes, then open the link above.

To change anything later: open the repo, click `index.html`, click the pencil, edit, commit.

---

## Notes

**Nothing is sent anywhere.** No server, no form handler, no analytics. The list sits in the
participant's own browser so they can close the tab and come back to it. You see it only when
they send you the file.

**The paste box takes most formats.** Tab, comma or semicolon separated. It finds the email
address wherever it sits on the line, picks up the relationship if it recognises the word
("line manager", "peer", "direct report", "skip level" and similar all work), and treats the
rest as the name. Outlook's `Name <address@domain>` form works too.

**Duplicate and malformed emails are flagged** as they type, with the field outlined, so a
typo does not turn into an invite that never arrives.

**The page is set to `noindex`** and will not appear in search results.

---

## Tested

Paste import across five formats including Arabic names and Outlook angle brackets; group
counting and suppression warnings; duplicate and invalid email detection; CSV quoting, encoding,
sort order and line endings; persistence across a reload; desktop and phone layouts. No console
errors.
