# Why Claude Feels Better Than Codex for Coding (and how i fixed it...)

I've been using both Claude Code and OpenAI's Codex CLI daily for about a month now. Codex is genuinely more capable in a lot of ways. It reasons deeper, it catches things Claude misses, and it can handle massive multi-file tasks without losing the plot.

But I kept coming back to Claude. Something about Codex just felt... off. Not bad. Just not fun. I couldn't figure out why until I sat down and actually studied my own conversation histories with both tools.

I did find some craaazy stuff.

---

## what i did

I pulled up my saved sessions, six Claude Code sessions where I built an Electron dictation app (Whisper.cpp, push-to-talk, the whole thing), and a long Codex session where I set up MCP tooling for a Next.js project.

Same user (me). Same kind of work. Very different experiences.

## what codex does well?

I want to be clear: Codex is not a worse tool. In my session, it ran 69 (lol) shell commands, cross-referenced documentation for five different IDEs, built a working npm package with tests, and proactively found a TLS issue on my production domain that I didn't even know about. That's impressive. Claude didn't do anything like that in my sessions.

So why did I name this Git "Why Claude Feels Better"?

## It's the Loop

The biggest difference isn't intelligence. It's how the two tools structure the back-and-forth.

With Claude, I'd say something like "it flickers, fix it." and Claude would fix the flickering, tell me what it did in one sentence, and wait. I'd test it, say what's next, and we'd keep going. It felt like pair programming ( we help each other proactively ). Fast cycles. Build, break, fix, build.

With Codex, I'd send a message and then wait. Codex would go off on its own, it'd read docs, run commands, reason through approaches and come back minutes later with a finished deliverable and a structured report. The report was good. But I was just sitting there the whole time.

And when Codex shipped something broken, i never bothered digging into it, it was easier to just revert and switch to Claude ( i dont want to wait another 30 minutes ).

Claude made me feel like I was building something. Codex made me feel like I was waiting for something.

## the personality thing

codex's default configuration strips out basically all personality. no casaul greetings, no emojis, no natural language quirks. Every response reads like a well written status update:

Codex > "I found your existing MCP route in src/app/api/mcp/route.ts and your docs already reference Cursor/Windsurf/Claude. I'm now reading your current mcp-setup docs so the package and config I add match your actual implementation and API shape."

its informative. its also something a CI pipeline would write if it could talk.

claude isn't exactly warm either, but there's a person in there somewhere. When I told it to "lock in" and build it "NOW" it just started building. No preamble. No plan. Files started appearing. that matched my energy. 

## Visible Struggle

this one surprised me. Claude's sessions were objectively messier. It tried three different hotkey libraries before finding one that worked. It had an FFmpeg race condition that took multiple rounds to fix. It never delivered the exact keyboard shortcut I asked for.

But I could see all of that happening. When something broke, Claude said "that crashed, trying X instead." when FFmpeg was writing empty files, Claude said "it's getting killed before it flushes" and fixed the shutdown sequence. The failures were transparent.

Codex's failures were silent. Commands would fail and it would just retry with a different approach. clean and professional, but I never got that moment of "oh it broke, oh wait it's fixed now! nice." The wins didn't feel earned because I never saw the fight.

The best moment in all my Claude sessions was when speech-to-text finally worked after four rounds of debugging. I typed "AMAZING" in all caps. That moment only existed because Id been in the trenches watching it fail.

## The Fix

So I wrote a system prompt to reshape how Codex works. Not to make it "be Claude".. I just addressed the specific problems:

**The batch problem:** I told it to work iteratively. Do one thing, show what you did, wait for my reaction. Never go off and do five things autonomously then present a report.

**The narration problem:** I told it to stop saying "I'm now reading..." and "I'm going to..." just do it and show results. Past tense over future tense. Results over intentions.

**Vaguity and intend:** I told it i might be vague with my prompt, trying to convey an intent, your job is to understand it. Gather context smartly try to find out what i wanted to be done.

**The personality problem:** I removed the restrictions on natural language. Allowed greetings, casual tone, energy matching. Write like a human.

**The over-engineering problem:** I told it to make the smallest change that fixes the problem. No refactoring nearby code, no adding abstractions, no "improving" things that weren't broken.

## fefore and after

here's a real example. Same bug, a toggle mode that required two taps to stop recording when it should only need one.

**Without the prompt**, Codex produced a 30-line diff. it ripped out the entire double-tap detection system, introduced a new state variable called `heldToRecord`, changed the tray icon from double-click to single-click (which nobody asked for), and rewrote the key-up handler. It worked, but it touched way more than it needed to and changed behavior I didn't complain about.

**With the prompt**, the fix was four lines:

```js
-          if (!toggleModeActive && isRecording) {
+          if (toggleModeActive && isRecording) {
+            toggleModeActive = false;
+            stopRecording();
+          } else if (!toggleModeActive && isRecording) {
               stopRecording();
```

That's it. If toggle mode is active and you're recording, one tap stops it. The explanation was one sentence: "toggle mode currently only exits on another double-tap, which is why stopping needs two taps again."

Same model. Same codebase. Same bug. Completely different approach. The prompt turned a refactor into a fix.
