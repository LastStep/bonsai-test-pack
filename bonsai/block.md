<!--
bonsai/block.md: the test pack's part of the instruction block (file kind block; Bonsai spec sections 5 and 6).
Purpose: Bonsai's engine writes the text after this comment into the linked project's CLAUDE.md, inside the marked
region Bonsai owns there. This comment is the file's documentation and stays in the pack: the engine writes
everything after the comment's closing line, never the comment itself.
When to use: read by bonsai init and bonsai update; never edited in a project (an edit inside the region is a
conflict at the next update that changes it). It is the same at every commit (A to D).
Fields: none; plain markdown, one line here. Example of what lands in CLAUDE.md: the line below.
-->
The test pack is linked (a Bonsai test fixture): its role is test-pack:marker and its skill is test-pack:hello.
