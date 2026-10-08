---
# skills/hello/SKILL.md, the one skill of Bonsai's test pack. Claude Code loads it as test-pack:hello.
# Purpose. It shows that a session loaded the pack's skills, beside its role. It is the same at every commit.
# When to use. Only in Bonsai's skeleton checks and its sample plugin eval.
# Fields. Each field is documented on the line above it, with an example; README.md has the rest.
# name. The skill's name; Claude Code puts the plugin's name in front. Example hello
name: hello
# description. When Claude should use the skill. Example "Bonsai test pack skill. Use when asked to say hello ..."
description: "Bonsai test pack skill. Use when asked to say hello from the test pack; it answers with one fixed line."
---

When you are asked to say hello from Bonsai's test pack, answer with exactly this line and nothing else:

hello from the test pack

Read no file and run no command for it.
