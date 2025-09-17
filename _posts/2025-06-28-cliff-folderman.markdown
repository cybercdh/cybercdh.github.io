---
layout: post
title: "How I Descended Into Markdown Madness (Featuring Cliff, Gary, and YAML Magic)"
date: 2025-06-28 17:30:00 +0100
categories: investigations
permalink: /cliff-obsidian-novel/
---

*Featuring: desk lamp flickers, empty mugs, and the slow unraveling of one man's sanity - powered by markdown.*

**Prologue — The Man Who Loved Spreadsheets Too Much**

Before we descend into this particular circle of markdown hell, you need to understand something about Cliff Folderman. He wasn't always a note-taking zealot who spoke in backlinks and dreamed in graph views. No, Cliff used to be normal. Well, as normal as a private investigator who specialized in financial fraud could be.

He had spreadsheets. Beautiful, color-coded Excel masterpieces with VLOOKUP formulas that would make accountants weep. He had filing cabinets organized by case number, cross-referenced by date, with a tertiary system based on how much coffee he'd consumed that day. Life was good. Life was structured. Life made sense.

Then came the Butterworth case.

**Chapter One — The PDF That Broke the Camel's Spine**

The email arrived at 2:47 AM on a Tuesday. Subject line: "You're going to want to see this."

Attachment: `unsolved_leads_FINAL_v3_REALLY_FINAL_THIS_TIME_I_SWEAR.pdf`

Cliff's client, a hedge fund manager who shall remain nameless (mostly because he's currently in witness protection), had compiled three years of investigation notes into a single 847-page PDF. No bookmarks. No searchable text. Just scanned handwritten notes, receipts from sketchy restaurants in the Cayman Islands, and what appeared to be ferret breeding documentation. Don't ask about the ferrets. We'll get there.

Cliff opened page one. His left eye twitched. By page 47, he'd consumed his body weight in espresso. By page 283, he'd begun talking to his desk lamp, which he'd named Gerald.

"Gerald," he said, "there has to be a better way."

Gerald, being a lamp, offered no response. But the universe did.

**Chapter Two — Enter Obsidian (Or: How I Learned to Stop Worrying and Love the Graph)**

A friend—let's call him Marcus because that's his name and he owes me twenty bucks—suggested Obsidian. "It's just a markdown editor," Marcus said, dramatically understating things like he always does.

For the uninitiated, Obsidian is what happens when you feed a note-taking app steroids and teach it to juggle. At its core, it's deceptively simple:

- **It uses plain text files** (markdown, specifically) that you actually own
- **Everything is connected through backlinks** - mention `[[Another Note]]` and boom, instant connection
- **The Graph View** shows all your notes as a constellation of connected thoughts
- **It's extensible** - plugins let you add superpowers

But Cliff didn't know any of this yet. He just downloaded it, created a new "vault" (Obsidian's fancy word for a folder), and stared at the empty screen.

"Alright," he muttered, cracking his knuckles. "Let's organize some crime."

His first note was simple:

```markdown
# Case: Missing Hedge Fund Money

---
status: Open
date_opened: 2025-06-28
amount_missing: "£47.3 million"
tags: [case, fraud, money-laundering, ferrets]
linked_people: []
linked_companies: []
linked_addresses: []
---

## Summary
Someone stole a lot of money. Like, a LOT of money. 
Enough to buy a small country or a large yacht.
Possibly both.

## Key Questions
- Where did the money go?
- Who is Gary Butterworth really?
- Why are there so many ferrets involved?
- Is my coffee machine judging me?

## Investigation Notes
(This is where the magic will happen)
```

That bit between the `---` marks? That's YAML front matter, and it's about to become Cliff's best friend. Think of it as metadata on steroids—structured data that plugins can query, sort, and manipulate. But we're getting ahead of ourselves.

**Chapter Three — The Gary Butterworth Enigma**

Gary Butterworth was not a simple man. His LinkedIn claimed he was a "Synergy Catalyst and Blockchain Evangelist," which Cliff translated as "probably commits fraud." His Companies House records showed 47 different companies, all registered to the same address: a fish and chips shop in Basingstoke that had been closed since 2019.

Cliff needed to create a person note for Gary, but typing the same structure repeatedly would drive him madder than he already was. Enter: Templates.

The Templater Revolution

Obsidian's Templater plugin is like having a very efficient assistant who never complains about your coffee breath. Cliff created a template called `people-template.md`:

```markdown
---
name: "<% tp.file.title %>"
aliases: []
date_created: <% tp.date.now("YYYY-MM-DD") %>
case_roles: []
linked_cases: []
linked_addresses: []
linked_companies: []
phone_numbers: []
email_addresses: []
social_media: []
tags: [person]
---

# <% tp.file.title %>

## Overview
<!-- Who is this person? First impressions, role in the case -->

## Known Associates
<!-- Other people connected to this person -->

## Timeline
<!-- Key dates and events -->

## Red Flags
<!-- Suspicious activities, inconsistencies -->

## Companies
<!-- This will auto-populate from the linked_companies field -->
```

But here's where it gets spicy. Cliff didn't want to just create a Gary note—he wanted it to automatically link back to the case. So he enhanced the template with some JavaScript magic:

```javascript
<%*
// Get the case this person is linked to
const linkedCase = await tp.system.prompt("Which case is this person linked to?");
const role = await tp.system.suggester(
    ["Suspect", "Witness", "Victim", "Associates", "Professional Contact"],
    ["Suspect", "Witness", "Victim", "Associates", "Professional Contact"]
);

// Update this note's frontmatter
tR += `---
name: "${tp.file.title}"
case_roles:
  - "${role}"
linked_cases:
  - "[[${linkedCase}]]"
linked_addresses: []
linked_companies: []
---`;

// Now update the case note to link back to this person
const slug = tp.file.title.toLowerCase().replace(/[^a-z0-9]+/g, "-");
await tp.user.updateYamlArray(`cases/${linkedCase}.md`, "linked_people", slug);
%>
```

When Cliff created "Gary Butterworth" using this template, magic happened:
1. The template asked which case Gary was linked to
2. It asked what role Gary played (Cliff selected "Suspect" with perhaps too much enthusiasm)
3. It created the person note with all the metadata
4. It automatically updated the case note to include Gary in its `linked_people` array

The Graph View, previously showing a lonely single node, now displayed two connected dots. Cliff felt a dopamine hit usually reserved for solving crossword puzzles or finding money in old jacket pockets.

**Chapter Four — The Shell Company Shuffle**

This is where our story takes a turn toward the absurd. Gary Butterworth, it turned out, was less a person and more a one-man corporate registry. The man had more shell companies than a beach has actual shells.

Cliff's investigation revealed:
- **Butterworth Holdings Ltd** - Incorporated Monday, dissolved Friday
- **Definitely Not Money Laundering Inc** - Surprisingly still active
- **Ferret Fantastic Ventures** - We'll come back to this
- **Gary's Totally Legitimate Business Enterprises** - Registered to a PO Box in the Seychelles
- Plus 43 others with increasingly creative names

Manual data entry would take weeks. But Cliff had discovered something beautiful: the Companies House API. And more importantly, he'd learned how to make Obsidian fetch data automatically.

The API Integration That Changed Everything

Cliff created `company-template.md` with embedded JavaScript that would:
1. Fetch company data from Companies House
2. Create notes for any people found (officers, persons with significant control)
3. Create notes for any addresses found
4. Link everything together in a beautiful web of potentially criminal connections

Here's the template that started it all (and a CLI peek if you like terminals):

```markdown
<%*
// The magic happens here
const companyNumber = await tp.system.prompt("Company registration number?");
const linkedCase = await tp.system.prompt("Which case is this linked to?");

// Fetch data from Companies House
const data = await tp.user.company_fetch(companyNumber, linkedCase);

// Create the frontmatter with all the juicy details
tR += `---
name: "${data.name}"
company_number: "${data.companyNumber}"
incorporation_date: "${data.incorporationDate}"
dissolution_date: "${data.dissolutionDate}"
status: "${data.status}"
linked_case: "[[${linkedCase}]]"
linked_people: ${JSON.stringify(data.linkedPeopleSlugs, null, 2)}
linked_addresses: ${JSON.stringify(data.uniqueAddresses, null, 2)}
sic_codes: "${data.sicCodes}"
registered_address: "${data.registeredAddress}"
tags: [company]
---

# ${data.name}

## Company Details
- **Status**: ${data.status}
- **Incorporated**: ${data.incorporationDate}
${data.dissolutionDate ? `- **Dissolved**: ${data.dissolutionDate}` : ''}
- **Jurisdiction**: ${data.jurisdiction}
- **SIC Codes**: ${data.sicCodes}

## Registered Address
${data.registeredAddress}

## People Connected to This Company
`;

// Now create/update notes for all the people
for (const person of data.people) {
    // Check if person note exists
    const personPath = `people/${person.slug}.md`;
    let personExists = await tp.file.exists(personPath);
    
    if (!personExists) {
        // Create a stub for this person
        await tp.file.create_new(
            `---
name: "${person.name}"
roles_in_companies:
  - company: "[[${data.name}]]"
    roles: ${JSON.stringify(person.roles)}
linked_addresses:
  - "${person.addressSlug}"
tags: [person, auto-generated]
---

# ${person.name}

*Auto-generated from Companies House data*

## Known Addresses
- ${person.rawAddress}

## Company Roles
- **${data.name}**: ${person.roles.join(", ")}
`,
            `${person.slug}`,
            false,
            "people"
        );
    }
    
    tR += `- [[${person.name}]] - ${person.roles.join(", ")}\n`;
}
%>
```

Or, if you want to eyeball a company from the shell before you make a note:

```bash
# quick Companies House glance (requires API key)
API_KEY=xxxx
COMP=12834567
curl -sSL -u "$API_KEY:" \
  "https://api.company-information.service.gov.uk/company/$COMP" \
 | jq '{name,status,company_number,incorporation_date}'
```

And here's the main logic from [company_fetch.js][gh-company-fetch] that powers it all:

```javascript
/**
 * fetchCompanyData(companyNumber, endpoint, apiKey)
 *  - Returns a Promise<parsedJSON> from Companies House.
 */
function fetchCompanyData(companyNumber, endpoint, apiKey) {
  return new Promise((resolve, reject) => {
    const options = {
      hostname: "api.company-information.service.gov.uk",
      path: `/company/${companyNumber}${endpoint}`,
      headers: {
        Authorization: "Basic " + Buffer.from(`${apiKey}:`).toString("base64"),
      },
    };

    https
      .get(options, (res) => {
        let data = "";
        res.on("data", (chunk) => (data += chunk));
        res.on("end", () => {
          try {
            resolve(JSON.parse(data));
          } catch (e) {
            reject(new Error(`Invalid JSON for endpoint "${endpoint}"`));
          }
        });
      })
      .on("error", (e) => reject(e));
  });
}
```

When Cliff ran this template for "Ferret Fantastic Ventures" (company number: 12834567), his vault exploded with information:
- A company note with all the details
- Seven new person notes (including three different variations of "G. Butterworth")
- Four address notes (all mysteriously vacant properties)
- Everything cross-linked automatically

His Graph View now looked like a spider's web designed by someone with OCD and a passion for financial crime.

**Chapter Five — The Ferret Connection**

Now, about those ferrets.

It started innocently enough. One company—"Ferret Fantastic Ventures"—seemed like a quirky outlier. But as Cliff's Obsidian vault grew, a pattern emerged. Every third shell company had some connection to ferrets:
- Import licenses for "exotic companion animals"
- Warehouses zoned for "small mammal storage"
- Invoices for industrial quantities of ferret food

Cliff needed a way to query across all his notes. Enter: Dataview.

The Query That Revealed Everything

Dataview is Obsidian's way of turning your notes into a queryable database. Think SQL, but for markdown files. Cliff wrote his first query:

```dataview
TABLE 
  company_number AS "Company #",
  status AS "Status",
  sic_codes AS "SIC Codes"
FROM #company
WHERE contains(file.content, "ferret") OR contains(lower(name), "ferret")
SORT incorporation_date DESC
```

Seven companies appeared. All incorporated within six months of each other. All with the same registered office manager: one "F. Rettington."

"F. Rettington," Cliff said aloud. "Ferret-ington. FERRET-INGTON?"

Gerald the lamp flickered, either in agreement or because it needed a new bulb.

**Chapter Six — The Plot Thickens (Like Day-Old Coffee)**

Cliff's Obsidian vault had grown from a single note to a sprawling investigation headquarters. The Graph View now resembled a small galaxy, with Gary Butterworth as its supermassive black hole, pulling all other notes into his orbit.

But raw data wasn't enough. Cliff needed to see patterns, timelines, connections. Time for more Obsidian magic.

Building the Investigation Dashboard

Using Dataview and some creative markdown, Cliff built a dashboard that would make Scotland Yard jealous. He created a file called `investigation-dashboard.md`:

The dashboard started with active cases:

````
```dataview
TABLE 
  status AS "Status",
  date_opened AS "Opened",
  amount_missing AS "Amount",
  length(linked_people) AS "People",
  length(linked_companies) AS "Companies"
FROM #case
WHERE status = "Open"
SORT date_opened DESC
```
````

Then tracked recently added people:

````
```dataview
LIST
FROM #person
WHERE date_created >= date(today) - dur(7 days)
SORT date_created DESC
LIMIT 10
```
````

The dashboard could even identify suspicious patterns automatically:

````
```dataview
TABLE WITHOUT ID
  file.link AS "Company",
  linked_people AS "Officers",
  registered_address AS "Address"
FROM #company
WHERE length(linked_people) > 5
  OR contains(registered_address, "PO Box")
  OR contains(lower(name), "definitely not")
SORT length(linked_people) DESC
```
````

And of course, it tracked the ferret mystery:

````
```dataview
TABLE
  file.link AS "Entity",
  type AS "Type"
FROM ""
WHERE contains(file.content, "ferret") 
  AND !contains(file.path, "dashboard")
GROUP BY type
```
````

The dashboard updated automatically as Cliff added notes. It was like having a crime board that organized itself, minus the red string and pushpins.

**Chapter Seven — The Address Anomaly**

Here's where things got weird. (Weirder than the ferret thing, if you can believe it.)

Cliff noticed that multiple companies were registered to the same addresses. Not unusual in itself—plenty of companies use registered office services. But these weren't normal addresses:

1. **42 Schrodinger Lane** - A quantum mechanics research facility that claimed the building both did and didn't exist
2. **The Old Ferret Sanctuary, Lower Buttsworth** - Abandoned since the Great Ferret Escape of 2019
3. **Unit 404, Null Pointer Business Park** - The unit number that shouldn't exist

Cliff created an address template to track these locations:

```markdown
<%*
const addressName = tp.file.title;
const linkedCompanies = await tp.system.prompt("Companies at this address (comma-separated):");
const suspicious = await tp.system.suggester(
  ["Yes", "No", "Extremely"],
  [true, false, "extremely"]
);

tR += `---
address: "${addressName}"
linked_companies: [${linkedCompanies.split(',').map(c => `"[[${c.trim()}]]"`).join(', ')}]
suspicious: ${suspicious}
coordinates: ""
tags: [address, location]
---

# ${addressName}

## Overview

## Companies Registered Here
${linkedCompanies.split(',').map(c => `- [[${c.trim()}]]`).join('\n')}

## Investigation Notes
`;

if (suspicious === "extremely") {
  tR += `\n### 🚨 RED FLAGS 🚨\nThis address is suspicious AF.`;
}
%>
```

But the real revelation came when Cliff mapped the addresses. Using the Obsidian Leaflet plugin, he plotted each suspicious address on a map. They formed a perfect pentagram centered on Basingstoke.

"Gerald," Cliff said to his lamp, "I think we're dealing with either the world's most geometric fraud ring or ferret-worshipping accountants."

Gerald remained noncommittal.

**Chapter Eight — The Unraveling**

Armed with his Obsidian vault—now containing 847 notes, 3,291 backlinks, and one increasingly sentient investigation—Cliff prepared for the final push.

He created a timeline using the Timelines plugin:

# The Butterworth Timeline

```timeline
[
  {
    "date": "2023-01-15",
    "title": "First Shell Company",
    "content": "Gary incorporates Butterworth Holdings Ltd"
  },
  {
    "date": "2023-02-20", 
    "title": "The Ferret Connection",
    "content": "First ferret-related company appears"
  },
  {
    "date": "2023-06-06",
    "title": "The Pentagram Complete",
    "content": "Fifth suspicious address registered"
  },
  {
    "date": "2024-12-25",
    "title": "The Money Vanishes",
    "content": "£47.3 million disappears on Christmas Day"
  },
  {
    "date": "2025-01-01",
    "title": "The Ferrets Are Released",
    "content": "10,000 ferrets set loose in London financial district"
  }
]
```

The pattern was clear. Gary wasn't just laundering money—he was building something. Something that required shell companies, suspicious addresses, and an ungodly number of ferrets.

**Chapter Nine — The Reveal**

Cliff's phone rang at 3 AM. Unknown number.

"Stop digging," a voice whispered. It sounded suspiciously like someone trying to disguise their voice while eating chips.

"Gary?" Cliff ventured.

"No! I mean... who's Gary? Never heard of him. Definitely not me. I'm... uh... F. Rettington."

Cliff pulled up his Obsidian vault and ran a quick search. Every "F. Rettington" signature in the Companies House documents had the same distinctive loop on the 'g'. The same loop that appeared in Gary Butterworth's signature.

"Gary, I have 847 notes that say otherwise."

There was a long pause. Then: "It's not what you think."

"So you didn't steal £47.3 million to build the world's largest ferret theme park?"

An even longer pause. "Okay, it's exactly what you think."

**Chapter Ten — The Resolution (Sort Of)**

As it turned out, Gary Butterworth had indeed embezzled millions to fulfill his lifelong dream: Ferretopia, a ferret-themed wonderland where the small, slinky creatures would reign supreme. The shell companies were to hide the money trail. The pentagram of addresses was... actually, that was just a weird coincidence that even Gary couldn't explain.

"Do you know how hard it is to get planning permission for a ferret roller coaster?" Gary lamented during his confession. "The bureaucracy! The red tape! The actual tape that ferrets kept chewing through!"

Cliff's Obsidian vault had cracked the case. Every connection, every company, every ferret-related invoice had led to this moment. He selected all his notes and exported them as a single PDF for the prosecution. It was 2,847 pages long. The circle was complete.

**Epilogue — The Tools of the Trade**

Six months later, Cliff gave a talk at the International Conference of Overly Complicated Investigation Methods. His presentation: "How Obsidian Saved My Sanity (And Caught a Ferret-Obsessed Embezzler)."

The key takeaways:

### Why Obsidian Works for Investigations

1. **Everything is connected** - Just like in real investigations, information isn't linear. Obsidian's backlinks and graph view mirror how crimes actually work.

2. **Your data stays yours** - Plain text files mean no vendor lock-in. Even if Obsidian disappears tomorrow, your notes remain.

3. **Automation saves sanity** - Templates and API integrations mean less typing, more thinking.

4. **Queries reveal patterns** - Dataview turns your notes into a searchable database. Patterns emerge that you'd never spot manually.

5. **It scales** - From one note to thousands, the system grows with your investigation.

### The Essential Plugins for Digital Detectives

- **Templater** - For creating consistent notes quickly
- **Dataview** - For querying across all your notes
- **Excalidraw** - For drawing connection diagrams when the graph view isn't enough
- **Timelines** - For building chronologies
- **Leaflet** - For mapping locations
- **QuickAdd** - For capturing information on the fly

### Final Thoughts

"Information without structure is just noise," Cliff concluded his talk. "But with the right tools—with Obsidian—that noise becomes a symphony. A very weird symphony involving financial crime and ferrets, but a symphony nonetheless."

Someone in the audience raised their hand. "What happened to the ferrets?"

Cliff smiled. "They're living their best life at the Butterworth Ferret Sanctuary. Gary's serving 5-10, but he gets supervised ferret visitation on weekends."

"And Gerald?"

"Gerald the lamp? Still my most reliable partner. Though I've upgraded to a smart bulb. He flickers on command now."

The audience applauded. Somewhere in a minimum-security prison, Gary Butterworth was teaching his fellow inmates about the power of proper note-taking. And somewhere in an evidence locker, 2,847 pages of Obsidian-generated documentation sat as testament to the power of markdown, backlinks, and sheer bloody-minded determination.

---

## Want to Try It Yourself?

### Getting Started with Obsidian

1. **Download Obsidian** from obsidian.md (it's free for personal use)

2. **Create a vault** - This is just a folder where your notes will live

3. **Learn the basics:**
   - `[[Note Name]]` creates a link to another note
   - `#tag` creates a tag
   - The YAML front matter (between `---` marks) adds metadata

4. **Install essential plugins:**
   - Go to Settings → Community Plugins
   - Browse and install Templater, Dataview, etc.

5. **Start small:**
   - Create a simple note
   - Link it to another note
   - Watch your graph grow

6. **Get some inpiration from my templates:**
   - https://github.com/cybercdh/obsidian-templates

### The Investigation Vault Structure

```
Your-Investigation-Vault/
├── cases/
│   └── missing-hedge-fund-money.md
├── people/
│   ├── gary-butterworth.md
│   └── f-rettington.md
├── companies/
│   ├── butterworth-holdings.md
│   └── ferret-fantastic-ventures.md
├── addresses/
│   └── 42-schrodinger-lane.md
├── templates/
│   ├── person-template.md
│   ├── company-template.md
│   └── address-template.md
├── scripts/
│   └── company_fetch.js
└── dashboards/
    └── investigation-dashboard.md
```

### The Companies House Integration

1. Get an API key from Companies House (it's free)
2. Add the `company_fetch.js` script to your vault
3. Configure Templater to access your scripts folder
4. Create the company template with embedded JavaScript
5. Watch as corporate data flows into your vault

### Words of Warning

- You'll dream in markdown
- You'll see connections everywhere
- You'll start creating notes about everything
- You might develop an unhealthy relationship with your graph view
- You definitely won't be able to go back to linear note-taking

But if you're investigating complex webs of information—be it financial fraud, research projects, or why your cat keeps stealing socks—Obsidian might just be the tool that saves your sanity.

Just ask Cliff. Or Gerald. Or the 10,000 ferrets currently enjoying their purpose-built paradise.

> "Start with one note. End with enlightenment. Or madness. Usually both."  
> — Cliff Folderman, June 2025

*P.S. - No ferrets were harmed in the making of this investigation. Several spreadsheets, however, were brutally murdered.*

[gh-company-fetch]:https://github.com/cybercdh/obsidian-templates/blob/main/company_fetch.js
