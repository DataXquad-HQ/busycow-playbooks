# Presentation Reframing: Pitch → Observation/Sharing Tone

## When to apply
User is giving a talk to a **peer or adjacent audience** (not a buyer). They want the talk to feel like a genuine sharing of experience, not a company pitch. Common triggers:
- Academic / university forums (e.g. 逢甲座談會)
- Industry exchanges where the audience are practitioners, not prospects
- User says "講得更像分享" / "不要像在 pitch 我們做的事" / "觀察的語氣"

## Core reframing moves

### 1. Self-introduction: company arc → personal journey
**Wrong frame:** "BusyCow 提供這些服務 / 我們的產品線是..."  
**Right frame:** "我們是怎麼走到這個題目的" — chronological company arc first (how the business evolved), then the speaker's role inside it.

Structure that works:
```
Company arc (what changed over time):
• Phase 1 → Phase 2 → Phase 3 (current experiment)
• Key external signal that caused each pivot

Speaker's role inside that arc:
• What they personally did / owned
• What gave them the vantage point to observe this topic
```

Ending line: something like "走這條路，我們也不是一開始就想清楚的。更多的時候是——試了，踩了坑，再試。今天講的，就是這段觀察。"

### 2. Section framing: narrow claim → broad observation
**Wrong:** "為什麼是中小企業？" (presupposes audience should care about your target market)  
**Right:** "每間公司都有可以讓 AI 介入的事——但規模不同，轉型難度差很多"

Rule: lead with the universal observation, then narrow to the specific context as supporting evidence — not the other way around.

### 3. Case studies: "我們做的事" → "幾個我們正在試的場景"
Subtle but important. "正在試" signals experiment-in-progress, not finished product pitch.

### 4. CTAs and closing: "找我們合作" → "歡迎聊聊 / 我們都還在路上"
The sharing tone ends with genuine openness, not a sales ask.

### 5. Examples in slides: use widely-known tools only
When listing AI tools as examples for a general audience, use only tools that have genuine mass recognition:
- ✅ ChatGPT, Claude, Perplexity, Cursor, GitHub Copilot
- ❌ Notion AI, NotebookLM, Copilot Studio, sector-specific tools

Rule of thumb: if you'd have to explain what it is, it's too niche for an observation-framed talk.

## Google Docs editing pattern for these changes
Use **targeted `replaceAllText`** for single-line anchors.  
Use **paragraph-index delete + insert** for block rewrites (e.g. replacing a full slide section).

Reliable block-rewrite sequence:
1. `replaceAllText` on a short unique anchor → converts it to a placeholder
2. Export doc as `text/plain` → find placeholder, check exact `\r\n` line endings
3. Read doc via Docs API → get paragraph `startIndex` / `endIndex` for the block
4. Single `batchUpdate` with `deleteContentRange` then `insertText` at same index
