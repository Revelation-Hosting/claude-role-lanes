# Setup without the plugin

Use this path if you do not install Claude Code plugins. The result is the same as `/lanes:setup`.

1. Clone this repository next to your project:

   ```bash
   git clone https://github.com/Revelation-Hosting/claude-role-lanes.git
   ```

2. Start Claude Code in **your project's** root directory.

3. Send this prompt, with the path changed to where you cloned the repository:

   ```
   Run the lanes setup procedure in ../claude-role-lanes/plugins/lanes/skills/setup/SKILL.md for this repository.
   SKILL_DIR is ../claude-role-lanes/plugins/lanes/skills/setup. Read SKILL.md and its reference/ files first,
   then follow the procedure step by step.
   ```

4. Answer the interview. Setup writes `.claude/lanes.yml` and the rendered files, verifies them, and asks before it
   creates GitHub labels or commits anything.

To re-render after you edit `.claude/lanes.yml`, or after you pull a newer version of this repository, send the same
prompt again. To report drift without writing, add "in check mode" to the prompt.
