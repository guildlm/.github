<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>GuildLM — Implementation Plan</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@300;400;500;600;700&family=JetBrains+Mono:wght@300;400;500&display=swap');

  :root {
    --bg: #0a0a0c;
    --bg2: #111116;
    --bg3: #18181f;
    --border: #222230;
    --gold: #C5A55A;
    --gold-dim: rgba(197,165,90,0.12);
    --gold-glow: rgba(197,165,90,0.06);
    --blue: #4a9eff;
    --blue-dim: rgba(74,158,255,0.1);
    --text: #e8e8f0;
    --muted: #6b6b80;
    --danger: #ff6b6b;
    --code-bg: #0f0f14;
  }

  * { margin: 0; padding: 0; box-sizing: border-box; }

  body {
    background: var(--bg);
    color: var(--text);
    font-family: 'Space Grotesk', sans-serif;
    font-size: 15px;
    line-height: 1.7;
  }

  /* ── HEADER ── */
  header {
    border-bottom: 1px solid var(--border);
    padding: 60px 0 50px;
    text-align: center;
    position: relative;
    overflow: hidden;
  }
  header::before {
    content: '';
    position: absolute;
    top: -80px; left: 50%; transform: translateX(-50%);
    width: 600px; height: 300px;
    background: radial-gradient(ellipse, rgba(197,165,90,0.08) 0%, transparent 70%);
    pointer-events: none;
  }
  .eyebrow {
    font-family: 'JetBrains Mono', monospace;
    font-size: 11px;
    letter-spacing: 0.22em;
    color: var(--gold);
    text-transform: uppercase;
    margin-bottom: 20px;
    opacity: 0.8;
  }
  h1 {
    font-size: clamp(36px, 5vw, 64px);
    font-weight: 700;
    letter-spacing: -0.03em;
    line-height: 1.1;
    margin-bottom: 14px;
  }
  h1 span { color: var(--gold); }
  .tagline {
    color: var(--muted);
    font-size: 17px;
    font-weight: 300;
    font-style: italic;
    letter-spacing: 0.01em;
  }

  /* ── LAYOUT ── */
  .container { max-width: 900px; margin: 0 auto; padding: 0 32px; }

  section {
    padding: 64px 0;
    border-bottom: 1px solid var(--border);
  }
  section:last-child { border-bottom: none; }

  .section-label {
    font-family: 'JetBrains Mono', monospace;
    font-size: 10px;
    letter-spacing: 0.2em;
    color: var(--gold);
    text-transform: uppercase;
    margin-bottom: 28px;
    opacity: 0.7;
    display: flex;
    align-items: center;
    gap: 12px;
  }
  .section-label::after {
    content: '';
    flex: 1;
    height: 1px;
    background: var(--border);
  }

  h2 {
    font-size: 28px;
    font-weight: 600;
    letter-spacing: -0.02em;
    margin-bottom: 20px;
    line-height: 1.25;
  }
  h3 {
    font-size: 16px;
    font-weight: 600;
    color: var(--gold);
    margin-bottom: 10px;
    letter-spacing: 0.01em;
  }
  p { color: #b0b0c0; margin-bottom: 16px; }
  p:last-child { margin-bottom: 0; }

  /* ── ARCHITECTURE DIAGRAM ── */
  .arch-diagram {
    background: var(--code-bg);
    border: 1px solid var(--border);
    border-radius: 12px;
    padding: 40px 32px;
    margin: 32px 0;
    overflow-x: auto;
  }
  .arch-diagram pre {
    font-family: 'JetBrains Mono', monospace;
    font-size: 12px;
    line-height: 1.8;
    color: #8888aa;
    white-space: pre;
  }
  .arch-diagram pre .highlight { color: var(--gold); }
  .arch-diagram pre .blue { color: var(--blue); }

  /* ── PHILOSOPHY TABLE ── */
  .philosophy-grid {
    display: grid;
    gap: 1px;
    background: var(--border);
    border: 1px solid var(--border);
    border-radius: 10px;
    overflow: hidden;
    margin: 28px 0;
  }
  .philosophy-row {
    display: grid;
    grid-template-columns: 180px 1fr;
    background: var(--bg2);
  }
  .philosophy-row:nth-child(odd) { background: var(--bg3); }
  .phil-key {
    padding: 14px 20px;
    font-family: 'JetBrains Mono', monospace;
    font-size: 12px;
    color: var(--gold);
    font-weight: 500;
    border-right: 1px solid var(--border);
    display: flex;
    align-items: center;
  }
  .phil-val {
    padding: 14px 20px;
    color: #a0a0b8;
    font-size: 14px;
    display: flex;
    align-items: center;
  }

  /* ── REVIEW BLOCKS ── */
  .review-block {
    background: rgba(197,165,90,0.05);
    border: 1px solid rgba(197,165,90,0.25);
    border-left: 3px solid var(--gold);
    border-radius: 0 8px 8px 0;
    padding: 20px 24px;
    margin: 16px 0;
  }
  .review-label {
    font-family: 'JetBrains Mono', monospace;
    font-size: 10px;
    letter-spacing: 0.15em;
    color: var(--gold);
    text-transform: uppercase;
    margin-bottom: 8px;
    opacity: 0.8;
  }
  .review-block p { color: #c0c0d0; margin: 0; font-size: 14px; }
  .review-block ul {
    margin: 8px 0 0 16px;
    color: #a0a0b8;
    font-size: 14px;
    line-height: 1.8;
  }

  /* ── REPO TABLE ── */
  .repo-table {
    width: 100%;
    border-collapse: collapse;
    margin: 24px 0;
    border: 1px solid var(--border);
    border-radius: 10px;
    overflow: hidden;
  }
  .repo-table thead tr { background: var(--bg3); }
  .repo-table th {
    text-align: left;
    padding: 12px 18px;
    font-family: 'JetBrains Mono', monospace;
    font-size: 10px;
    letter-spacing: 0.15em;
    color: var(--muted);
    text-transform: uppercase;
    border-bottom: 1px solid var(--border);
  }
  .repo-table td {
    padding: 13px 18px;
    font-size: 13px;
    border-bottom: 1px solid var(--border);
    vertical-align: top;
  }
  .repo-table tr:last-child td { border-bottom: none; }
  .repo-table tr:nth-child(even) td { background: rgba(255,255,255,0.015); }
  .mono { font-family: 'JetBrains Mono', monospace; color: var(--blue); font-size: 12px; }
  .badge {
    display: inline-block;
    padding: 2px 8px;
    border-radius: 4px;
    font-size: 10px;
    font-family: 'JetBrains Mono', monospace;
    letter-spacing: 0.05em;
  }
  .badge-core { background: var(--gold-dim); color: var(--gold); }
  .badge-guild { background: var(--blue-dim); color: var(--blue); }

  /* ── PHASES ── */
  .phase-list { display: flex; flex-direction: column; gap: 32px; margin: 32px 0; }
  .phase {
    display: grid;
    grid-template-columns: 100px 1fr;
    gap: 24px;
    position: relative;
  }
  .phase::before {
    content: '';
    position: absolute;
    left: 49px; top: 48px; bottom: -32px;
    width: 1px;
    background: var(--border);
  }
  .phase:last-child::before { display: none; }
  .phase-num {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 6px;
    padding-top: 4px;
  }
  .phase-circle {
    width: 36px; height: 36px;
    border-radius: 50%;
    border: 1.5px solid var(--gold);
    display: flex; align-items: center; justify-content: center;
    font-family: 'JetBrains Mono', monospace;
    font-size: 12px;
    color: var(--gold);
    background: var(--bg);
    flex-shrink: 0;
    position: relative;
    z-index: 1;
  }
  .phase-week {
    font-family: 'JetBrains Mono', monospace;
    font-size: 10px;
    color: var(--muted);
    text-align: center;
    line-height: 1.3;
  }
  .phase-body { padding-bottom: 4px; }
  .phase-title {
    font-size: 17px;
    font-weight: 600;
    margin-bottom: 6px;
    letter-spacing: -0.01em;
  }
  .phase-title .mono { font-size: 14px; }
  .phase-desc { color: #8888aa; font-size: 14px; margin-bottom: 12px; }

  /* ── CODE BLOCK ── */
  .code-block {
    background: var(--code-bg);
    border: 1px solid var(--border);
    border-radius: 8px;
    padding: 20px 22px;
    margin: 16px 0;
    overflow-x: auto;
  }
  .code-block pre {
    font-family: 'JetBrains Mono', monospace;
    font-size: 12px;
    line-height: 1.75;
    color: #7070a0;
  }
  .code-block pre .c { color: var(--gold); }
  .code-block pre .k { color: var(--blue); }
  .code-block pre .s { color: #80d080; }
  .code-block pre .m { color: #e08060; }
  .code-block pre .comment { color: #404060; }

  /* ── EXECUTION ORDER ── */
  .execution-flow {
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
    margin: 24px 0;
    align-items: center;
  }
  .flow-node {
    padding: 8px 16px;
    border-radius: 6px;
    font-size: 13px;
    font-weight: 500;
  }
  .flow-node-gold { background: var(--gold-dim); border: 1px solid rgba(197,165,90,0.3); color: var(--gold); }
  .flow-node-blue { background: var(--blue-dim); border: 1px solid rgba(74,158,255,0.3); color: var(--blue); }
  .flow-node-dark { background: var(--bg3); border: 1px solid var(--border); color: var(--muted); }
  .flow-arrow { color: var(--border); font-size: 18px; }

  /* ── SUCCESS TABLE ── */
  .success-table {
    width: 100%;
    border-collapse: collapse;
    margin: 24px 0;
    border: 1px solid var(--border);
    border-radius: 10px;
    overflow: hidden;
  }
  .success-table thead tr { background: var(--bg3); }
  .success-table th {
    text-align: left;
    padding: 12px 20px;
    font-family: 'JetBrains Mono', monospace;
    font-size: 10px;
    letter-spacing: 0.15em;
    color: var(--muted);
    text-transform: uppercase;
    border-bottom: 1px solid var(--border);
  }
  .success-table td {
    padding: 14px 20px;
    border-bottom: 1px solid var(--border);
    font-size: 14px;
  }
  .success-table tr:last-child td { border-bottom: none; }
  .success-table tr:nth-child(even) td { background: rgba(255,255,255,0.015); }
  .target { color: var(--gold); font-family: 'JetBrains Mono', monospace; font-size: 13px; }

  /* ── CHECKLIST ── */
  .checklist { list-style: none; display: flex; flex-direction: column; gap: 10px; margin: 24px 0; }
  .checklist li {
    display: flex;
    align-items: flex-start;
    gap: 12px;
    font-size: 14px;
    color: #a0a0b8;
    padding: 10px 16px;
    background: var(--bg2);
    border: 1px solid var(--border);
    border-radius: 7px;
  }
  .check-icon { color: var(--gold); font-size: 16px; flex-shrink: 0; line-height: 1.5; }

  /* ── BRAIN FLOW ── */
  .brain-flow {
    background: var(--code-bg);
    border: 1px solid var(--border);
    border-radius: 10px;
    padding: 28px;
    margin: 20px 0;
  }
  .brain-step {
    display: grid;
    grid-template-columns: 28px 1fr;
    gap: 14px;
    margin-bottom: 14px;
    align-items: start;
  }
  .brain-step:last-child { margin-bottom: 0; }
  .step-num-b {
    width: 24px; height: 24px;
    background: var(--gold-dim);
    border: 1px solid rgba(197,165,90,0.3);
    border-radius: 50%;
    display: flex; align-items: center; justify-content: center;
    font-family: 'JetBrains Mono', monospace;
    font-size: 10px;
    color: var(--gold);
    flex-shrink: 0;
    margin-top: 2px;
  }
  .step-body { color: #8080a8; font-size: 13px; line-height: 1.6; }
  .step-body strong { color: var(--text); font-weight: 500; }
  .step-body .tag {
    display: inline-block;
    background: var(--blue-dim);
    color: var(--blue);
    padding: 1px 7px;
    border-radius: 4px;
    font-family: 'JetBrains Mono', monospace;
    font-size: 11px;
    margin: 0 2px;
  }
  .step-body .tag-gold {
    background: var(--gold-dim);
    color: var(--gold);
  }
</style>
</head>
<body>

<!-- HEADER -->
<header>
  <div class="container">
    <div class="eyebrow">Implementation Plan · Open Source · Local-First</div>
    <h1>Guild<span>LM</span></h1>
    <div class="tagline">Specialist Models. Mastered.<br>One master per domain. One brain to coordinate them all.</div>
  </div>
</header>

<!-- BIG PICTURE -->
<section>
  <div class="container">
    <div class="section-label">Architecture</div>
    <h2>The Big Picture</h2>
    <p>Instead of one monolithic 500B model that tries to know everything, GuildLM trains dozens of small specialist models (3B–8B each), grouped into domain guilds. A central Brain classifies intent and routes every request to the right specialist.</p>

    <div class="arch-diagram">
      <pre>
                       ┌──────────────┐
                       │    USER      │
                       └──────┬───────┘
                              │
                       ┌──────▼───────┐
                       │  <span class="highlight">🧠 BRAIN</span>    │
                       │  (Planner)   │
                       │  Routes to   │
                       │  the right   │
                       │  guild       │
                       └──────┬───────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
 ┌──────▼──────┐      ┌──────▼──────┐      ┌──────▼──────┐
 │ <span class="highlight">⚔️  CODE</span>    │      │ <span class="highlight">⚖️  LEGAL</span>   │      │ <span class="highlight">🏥 MEDICAL</span>  │
 │   GUILD     │      │   GUILD     │      │   GUILD     │
 │             │      │             │      │             │
 │ Go SLM      │      │ Contract    │      │ Diagnosis   │
 │ React SLM   │      │ Compliance  │      │ Literature  │
 │ SQL SLM     │      │ Case Law    │      │ Drug Inter. │
 │ DevOps SLM  │      │ IP Law      │      │ Radiology   │
 └─────────────┘      └─────────────┘      └─────────────┘
        │                     │                     │
 ┌──────▼──────┐      ┌──────▼──────┐      ┌──────▼──────┐
 │ <span class="highlight">💰 FINANCE</span>  │      │ <span class="highlight">📚 EDUCATION</span>│      │ <span class="highlight">🎨 CREATIVE</span> │
 │   GUILD     │      │   GUILD     │      │   GUILD     │
 │             │      │             │      │             │
 │ Risk Anal.  │      │ Tutor       │      │ Copywriting │
 │ Trading     │      │ Curriculum  │      │ Storytelling│
 │ Compliance  │      │ Assessment  │      │ Translation │
 │ Tax         │      │ Research    │      │ Marketing   │
 └─────────────┘      └─────────────┘      └─────────────┘</pre>
    </div>

    <p><strong>The key distinction:</strong> the Brain doesn't answer anything itself. It just decides — <em>"Is this Go code? → Code Guild. Is this a contract? → Legal Guild."</em> Domain isolation means each SLM is sharper, smaller, and faster than any generalist alternative.</p>
  </div>
</section>

<!-- PHILOSOPHY -->
<section>
  <div class="container">
    <div class="section-label">Core Philosophy</div>
    <h2>Six Principles</h2>

    <div class="philosophy-grid">
      <div class="philosophy-row">
        <div class="phil-key">Guild System</div>
        <div class="phil-val">Every domain is a "guild" — containing multiple specialist SLMs</div>
      </div>
      <div class="philosophy-row">
        <div class="phil-key">Brain First</div>
        <div class="phil-val">The Brain identifies the task and routes it to the correct guild</div>
      </div>
      <div class="philosophy-row">
        <div class="phil-key">Domain Isolation</div>
        <div class="phil-val">Each SLM knows its own domain only — nothing else</div>
      </div>
      <div class="philosophy-row">
        <div class="phil-key">Composable</div>
        <div class="phil-val">Adding a new domain = adding a new guild. Nothing else changes.</div>
      </div>
      <div class="philosophy-row">
        <div class="phil-key">Local-First</div>
        <div class="phil-val">The entire system must be able to run on a single machine</div>
      </div>
      <div class="philosophy-row">
        <div class="phil-key">Open Source</div>
        <div class="phil-val">Everything is open source and community-driven</div>
      </div>
    </div>
  </div>
</section>

<!-- OPEN QUESTIONS -->
<section>
  <div class="container">
    <div class="section-label">User Review Required</div>
    <h2>Three Open Questions</h2>

    <div class="review-block">
      <div class="review-label">Scope</div>
      <p>We're starting with <strong>Code Guild (Go)</strong> as the first guild. But the core infrastructure — forge, anvil, crucible, sentinel — will be fully <strong>domain-agnostic</strong>. Adding Legal Guild later will only require a new config + dataset + LoRA. Is this approach correct?</p>
    </div>

    <div class="review-block">
      <div class="review-label">Brain Model</div>
      <p>Two options for the Brain:</p>
      <ul>
        <li>Use an existing general model (Qwen3-8B, Llama 3.1) — sufficient for Phase 1</li>
        <li>Train the Brain itself — specialized in task routing, intent classification, and guild selection</li>
      </ul>
      <p style="margin-top:10px;">Recommendation: use an existing model in Phase 1, then train our own Brain model later.</p>
    </div>

    <div class="review-block">
      <div class="review-label">GPU Infrastructure</div>
      <p>What will we use for training? (A100, consumer GPU, cloud?)</p>
    </div>
  </div>
</section>

<!-- REPO STRUCTURE -->
<section>
  <div class="container">
    <div class="section-label">Repository Structure</div>
    <h2>Updated Repo Layout</h2>

    <table class="repo-table">
      <thead>
        <tr>
          <th>Repository</th>
          <th>Purpose</th>
          <th>Type</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td><span class="mono">guildlm.github.io</span></td>
          <td>Landing page, docs, guild catalog</td>
          <td>—</td>
        </tr>
        <tr>
          <td><span class="mono">brain</span></td>
          <td>🧠 Central orchestrator — task routing, guild selection, multi-guild coordination</td>
          <td><span class="badge badge-core">Core</span></td>
        </tr>
        <tr>
          <td><span class="mono">forge</span></td>
          <td>Data pipeline — data collection, cleaning, instruction generation for <em>any domain</em></td>
          <td><span class="badge badge-core">Core</span></td>
        </tr>
        <tr>
          <td><span class="mono">anvil</span></td>
          <td>Training infra — SFT, DPO, LoRA, merge, export for <em>any model</em></td>
          <td><span class="badge badge-core">Core</span></td>
        </tr>
        <tr>
          <td><span class="mono">crucible</span></td>
          <td>Evaluation — benchmarks, testing, comparison for <em>any domain</em></td>
          <td><span class="badge badge-core">Core</span></td>
        </tr>
        <tr>
          <td><span class="mono">guild-code</span></td>
          <td>⚔️ First guild: Code SLMs (Go, React, SQL, DevOps) — configs, datasets, benchmarks</td>
          <td><span class="badge badge-guild">Guild</span></td>
        </tr>
        <tr>
          <td><span class="mono">guild-template</span></td>
          <td>📋 Boilerplate template for creating new guilds</td>
          <td>Template</td>
        </tr>
      </tbody>
    </table>

    <p><strong>Core tools never change</strong> — only new <code style="font-family:monospace;color:var(--blue)">guild-*</code> repos get added. Legal Guild, Medical Guild, Finance Guild — they all plug in without modifying the engine.</p>
  </div>
</section>

<!-- PHASES -->
<section>
  <div class="container">
    <div class="section-label">Implementation Phases</div>
    <h2>Seven Phases</h2>

    <div class="phase-list">

      <div class="phase">
        <div class="phase-num">
          <div class="phase-circle">1</div>
          <div class="phase-week">Week<br>1</div>
        </div>
        <div class="phase-body">
          <div class="phase-title">Foundation — Landing Page + Core Repos</div>
          <div class="phase-desc">Premium, dark-themed brand showcase. Not just a promo page — closer to a manifesto.</div>
          <p style="font-size:13px;color:#7070a8;">Hero · Problem statement · Guild showcase · Interactive architecture diagram · Roadmap · Open Source links. Each core repo initialized with README, Apache 2.0 license, GitHub Actions CI, contributing guidelines.</p>
        </div>
      </div>

      <div class="phase">
        <div class="phase-num">
          <div class="phase-circle">2</div>
          <div class="phase-week">Wks<br>2–4</div>
        </div>
        <div class="phase-body">
          <div class="phase-title">Data Pipeline — <span class="mono">forge</span></div>
          <div class="phase-desc">Domain-agnostic. Forge knows nothing about domains — only <code style="font-family:monospace;color:var(--blue)">sources/</code> and <code style="font-family:monospace;color:var(--blue)">plugins/</code> carry domain knowledge.</div>
          <div class="code-block"><pre>forge/
├── src/
│   ├── core/                    <span class="comment"># Domain-agnostic engine</span>
│   │   ├── discoverer.py        <span class="comment"># Data source discovery</span>
│   │   ├── downloader.py        <span class="comment"># Concurrent content fetcher</span>
│   │   ├── processor.py         <span class="comment"># Clean, dedup, validate</span>
│   │   ├── instruction_gen.py   <span class="comment"># Q&amp;A pair generation</span>
│   │   └── dataset_builder.py   <span class="comment"># Final JSONL/Parquet export</span>
│   ├── sources/                 <span class="comment"># Pluggable data sources</span>
│   │   ├── github.py
│   │   ├── arxiv.py
│   │   ├── legal_corpus.py
│   │   └── pubmed.py
│   └── plugins/                 <span class="comment"># Domain-specific processors</span>
│       ├── code/
│       └── text/
└── configs/</pre></div>
        </div>
      </div>

      <div class="phase">
        <div class="phase-num">
          <div class="phase-circle">3</div>
          <div class="phase-week">Wks<br>4–6</div>
        </div>
        <div class="phase-body">
          <div class="phase-title">Training Infrastructure — <span class="mono">anvil</span></div>
          <div class="phase-desc">Trains any model on any data. SFT, DPO, LoRA, merge, quantization (GPTQ, AWQ, GGUF). Per-guild training recipes in <code style="font-family:monospace;color:var(--blue)">configs/guilds/</code>.</div>
          <div class="code-block"><pre>anvil/
├── src/
│   ├── train.py                 <span class="comment"># SFT trainer (any model, any data)</span>
│   ├── dpo_train.py             <span class="comment"># DPO/RLHF trainer</span>
│   ├── merge.py                 <span class="comment"># LoRA merge + export (HF, GGUF, ONNX)</span>
│   └── quantize.py              <span class="comment"># GPTQ, AWQ, GGUF quantization</span>
└── configs/
    ├── base_models/             <span class="comment"># qwen3_8b, llama_8b, mistral_7b</span>
    ├── lora/                    <span class="comment"># default, qlora_consumer, high_rank</span>
    └── guilds/                  <span class="comment"># per-guild training recipes</span></pre></div>
        </div>
      </div>

      <div class="phase">
        <div class="phase-num">
          <div class="phase-circle">4</div>
          <div class="phase-week">Wks<br>5–7</div>
        </div>
        <div class="phase-body">
          <div class="phase-title">Evaluation — <span class="mono">crucible</span></div>
          <div class="phase-desc">Pluggable evaluators for any domain. Functional tests, factual accuracy, quality scoring, safety & hallucination detection, and LLM-as-judge. Docker-based sandboxes for code execution.</div>
        </div>
      </div>

      <div class="phase">
        <div class="phase-num">
          <div class="phase-circle">5</div>
          <div class="phase-week">Wks<br>7–9</div>
        </div>
        <div class="phase-body">
          <div class="phase-title">The Brain — <span class="mono">brain</span> 🧠</div>
          <div class="phase-desc">The most critical component. Understands user intent and routes to the correct guild. VRAM-aware scheduling, hot-swap LoRA adapters, Ollama / vLLM / local support.</div>
          <h3 style="margin-top:16px;">Brain Flow — Single Guild</h3>
          <div class="brain-flow">
            <div class="brain-step">
              <div class="step-num-b">→</div>
              <div class="step-body"><strong>User:</strong> "Find and fix the race condition in this Go code"</div>
            </div>
            <div class="brain-step">
              <div class="step-num-b">1</div>
              <div class="step-body"><strong>Brain.classify()</strong> → Domain: <span class="tag tag-gold">code</span> · Language: <span class="tag tag-gold">go</span> · Task: <span class="tag tag-gold">bug_fix</span> · Subtask: <span class="tag tag-gold">concurrency</span></div>
            </div>
            <div class="brain-step">
              <div class="step-num-b">2</div>
              <div class="step-body"><strong>Brain.route()</strong> → Guild: <span class="tag">code</span> → Specialist: <span class="tag">go_reviewer</span> (first pass) → <span class="tag">go_generator</span> (fix) → <span class="tag">go_reviewer</span> (verify)</div>
            </div>
            <div class="brain-step">
              <div class="step-num-b">3</div>
              <div class="step-body"><strong>Brain.execute()</strong> → Load go_reviewer → Analyze → Find race condition → Load go_generator → Generate fix → Run go test → Final review → Return to user</div>
            </div>
          </div>

          <h3 style="margin-top:20px;">Brain Flow — Multi-Guild</h3>
          <div class="brain-flow">
            <div class="brain-step">
              <div class="step-num-b">→</div>
              <div class="step-body"><strong>User:</strong> "Build a fintech API in Go, GDPR compliant"</div>
            </div>
            <div class="brain-step">
              <div class="step-num-b">1</div>
              <div class="step-body"><span class="tag tag-gold">Code Guild (Go)</span> → API skeleton</div>
            </div>
            <div class="brain-step">
              <div class="step-num-b">2</div>
              <div class="step-body"><span class="tag">Legal Guild (Compliance)</span> → GDPR requirements check</div>
            </div>
            <div class="brain-step">
              <div class="step-num-b">3</div>
              <div class="step-body"><span class="tag tag-gold">Code Guild (Go)</span> → Apply compliance requirements</div>
            </div>
            <div class="brain-step">
              <div class="step-num-b">4</div>
              <div class="step-body"><span class="tag tag-gold">Code Guild (Security)</span> → Security audit</div>
            </div>
            <div class="brain-step">
              <div class="step-num-b">5</div>
              <div class="step-body"><span class="tag">Legal Guild (Review)</span> → Final compliance review</div>
            </div>
          </div>
        </div>
      </div>

      <div class="phase">
        <div class="phase-num">
          <div class="phase-circle">6</div>
          <div class="phase-week">Wks<br>8–10</div>
        </div>
        <div class="phase-body">
          <div class="phase-title">First Guild — <span class="mono">guild-code</span></div>
          <div class="phase-desc">Starting with Go. Each specialist role has its own system prompt, dataset config, and training recipe. React, SQL, and DevOps follow in Phase 2 with the same structure.</div>
          <div class="code-block"><pre>guild-code/
├── go/
│   ├── dataset_config.yaml
│   ├── training_config.yaml
│   ├── benchmarks/
│   ├── tools/                   <span class="comment"># go test, lint, etc.</span>
│   └── prompts/
│       ├── generator.txt
│       ├── reviewer.txt
│       ├── optimizer.txt
│       └── security.txt
├── react/                       <span class="comment"># Phase 2</span>
├── sql/                         <span class="comment"># Phase 2</span>
└── devops/                      <span class="comment"># Phase 2</span></pre></div>
        </div>
      </div>

      <div class="phase">
        <div class="phase-num">
          <div class="phase-circle">7</div>
          <div class="phase-week">After</div>
        </div>
        <div class="phase-body">
          <div class="phase-title">Guild Template — <span class="mono">guild-template</span></div>
          <div class="phase-desc">Makes creating a new guild trivial. One command, then fill in the configs.</div>
          <div class="code-block"><pre><span class="comment"># Create a new guild</span>
gh repo create guildlm/guild-legal <span class="k">--template</span> guildlm/guild-template
cd guild-legal
<span class="comment"># Fill configs → run forge → train with anvil → test with crucible</span></pre></div>
        </div>
      </div>

    </div>
  </div>
</section>

<!-- EXECUTION ORDER -->
<section>
  <div class="container">
    <div class="section-label">Execution Order</div>
    <h2>Dependency Graph</h2>
    <p>Foundation and core infra run in parallel. Brain only starts once Forge and Crucible are ready. Everything converges at the first guild.</p>
    <div class="execution-flow">
      <span class="flow-node flow-node-gold">Phase 1 · Foundation</span>
      <span class="flow-arrow">→</span>
      <span class="flow-node flow-node-dark">Phase 2 · Forge</span>
      <span class="flow-arrow">+</span>
      <span class="flow-node flow-node-dark">Phase 3 · Anvil</span>
      <span class="flow-arrow">→</span>
      <span class="flow-node flow-node-dark">Phase 4 · Crucible</span>
      <span class="flow-arrow">→</span>
      <span class="flow-node flow-node-gold">Phase 5 · Brain</span>
      <span class="flow-arrow">→</span>
      <span class="flow-node flow-node-blue">Phase 6 · guild-code</span>
      <span class="flow-arrow">→</span>
      <span class="flow-node flow-node-dark">Phase 7 · Template</span>
    </div>
  </div>
</section>

<!-- VERIFICATION -->
<section>
  <div class="container">
    <div class="section-label">Verification Plan</div>
    <h2>Smoke Test + Success Criteria</h2>

    <h3>End-to-End Smoke Test</h3>
    <div class="code-block" style="margin-bottom:28px;">
      <pre>User → Brain → Code Guild (Go) → Generate code → Review → Test → Return</pre>
    </div>

    <h3 style="margin-bottom:16px;">Success Criteria</h3>
    <table class="success-table">
      <thead>
        <tr>
          <th>Metric</th>
          <th>Target</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td>Brain routing accuracy</td>
          <td class="target">&gt;95% correct guild selection</td>
        </tr>
        <tr>
          <td>Go HumanEval pass@1</td>
          <td class="target">&gt;70% (vs base ~55%)</td>
        </tr>
        <tr>
          <td>New guild setup time</td>
          <td class="target">&lt;1 day from template to working guild</td>
        </tr>
        <tr>
          <td>Local VRAM usage</td>
          <td class="target">&lt;16 GB for Brain + 1 Guild</td>
        </tr>
        <tr>
          <td>System latency</td>
          <td class="target">&lt;10s for single-step tasks</td>
        </tr>
      </tbody>
    </table>

    <h3 style="margin-bottom:16px; margin-top:28px;">Automated Tests</h3>
    <ul class="checklist">
      <li><span class="check-icon">✓</span> pytest for all Python code across all core repos</li>
      <li><span class="check-icon">✓</span> go test for the Go sandbox runner in crucible</li>
      <li><span class="check-icon">✓</span> GitHub Actions CI on every repository</li>
    </ul>
  </div>
</section>

<!-- READY TO GO -->
<section>
  <div class="container">
    <div class="section-label">Ready to Execute</div>
    <h2>Start Order</h2>
    <p>Once approved, execution proceeds in this sequence:</p>
    <ul class="checklist">
      <li><span class="check-icon">01</span> <code style="font-family:monospace;color:var(--blue)">guildlm.github.io</code> — Premium landing page (manifesto + architecture)</li>
      <li><span class="check-icon">02</span> <code style="font-family:monospace;color:var(--blue)">brain</code> — Create repo, scaffold core structure</li>
      <li><span class="check-icon">03</span> <code style="font-family:monospace;color:var(--blue)">forge</code> — Create repo, domain-agnostic pipeline</li>
      <li><span class="check-icon">04</span> <code style="font-family:monospace;color:var(--blue)">anvil</code> — Create repo, training infrastructure</li>
      <li><span class="check-icon">05</span> <code style="font-family:monospace;color:var(--blue)">crucible</code> — Create repo, evaluation framework</li>
      <li><span class="check-icon">06</span> <code style="font-family:monospace;color:var(--blue)">guild-code</code> — First guild, Go specialist</li>
      <li><span class="check-icon">07</span> <code style="font-family:monospace;color:var(--blue)">guild-template</code> — New guild boilerplate</li>
    </ul>
  </div>
</section>

</body>
</html>
