# Fast, Safe Websites for About $2 a Month: Plain English Version

This is a short, plain English version of the full guide by Chris Binnie. It uses short sentences and explains each technical word.

[Read the full guide](README.md) for all the details and code.

---

## What this guide is about

Chris built two websites: [chrisbinnie.com](https://www.chrisbinnie.com) and [chrisbinnie.co.uk](https://www.chrisbinnie.co.uk).

Both websites are:

- **safe**: they are hard to attack
- **fast**: they open very quickly
- **easy to find**: search engines like Google rank them well

They cost about $1 to $2 a month to run. This does not include the yearly cost of the domain name.

---

## How we know the websites are good

Free testing websites check other websites and give them a score.

| Test | What it checks | Score |
|---|---|---|
| SSL Labs | The secure connection (the padlock in your browser) | A+, the top grade |
| Mozilla Observatory | Security settings | 125 out of 100 (it gives bonus points) |
| Google PageSpeed | Speed | 100 out of 100 |
| Google PageSpeed | Accessibility (can everyone use it?) | 94 out of 100 |
| Google PageSpeed | Search engines | 100 out of 100 |

---

## The big idea: keep the website simple

Most websites are built with big systems like WordPress. These systems run programs every time someone visits. They also use lots of code written by other people.

More code means:

- more places for attackers to break in
- more work for the computer, so the page is slower

Chris's websites are **static**. A static website is a set of ready-made files. Nothing needs to run when you visit. The files are just sent to you.

Static websites work well for:

- personal websites
- guides and documents
- company information pages
- blogs

Static websites are not a good fit for:

- online shops with lots of stock
- social networks
- websites where you log in

---

## Where the website lives

The websites use two services from Amazon Web Services (**AWS**):

- **S3** stores the files.
- **CloudFront** sends the files to visitors. It keeps copies in many countries, so the files never travel far. It also protects the websites from floods of fake visits.

---

## How the websites stay safe

The websites send **security headers** with every page. A security header is a short instruction to your browser. For example, one header tells the browser: "Only run code that comes from this website."

The websites use nine security headers.

The websites also use no code from other people. So a problem in someone else's code cannot affect them.

---

## How the websites stay fast

- **The pages are small.** The chrisbinnie.com home page is about 23 KB. Many websites are 100 times bigger.
- **Files are squeezed before sending.** This is called **compression**. It makes text files much smaller on the way to you.
- **Pictures are made as small as possible** without looking worse.
- **The browser is told the size of each picture.** This stops the page jumping around while it loads.
- **Scripts wait their turn.** A script is a small program in the page. Here, scripts load without stopping the page from showing.

---

## How search engines find the websites

- Each page has a clear **title** and a short **description**. Search engines show these in their results.
- Each page includes **structured data**. This is hidden information that tells search engines what the page is about, such as the author's name.
- A **sitemap** lists every page on the website. Search engines use it to find pages.
- Speed and security also help a website rank well.

---

## Looking after the websites

Static websites need very little care. Chris suggests these checks.

**Every month:**

- Run the security and speed tests again.
- Check the padlock certificate is still valid. Amazon renews it for you.

**Every few months:**

- Update the words on the pages.
- Check the pictures are still as small as they can be.

**Every year:**

- Do a full security check.
- Look for new features from Amazon.

---

## One thing to watch out for

Browsers save copies of files to make websites faster next time. This is called **caching**.

If a file is saved for a long time and you then change it, visitors may keep seeing the old version. The fix is to give the file a new name each time it changes.

The full guide explains how to do this.

---

## A warning from Chris

The examples in the full guide are for learning. If security settings are set up wrongly, a website can stop working or become less safe. Test any change carefully before you use it.

---

[Read the full guide](README.md)
