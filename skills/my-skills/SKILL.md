---
name: my-skills
description: List all installed skills and plugins with their names, descriptions, and slash commands. Use when the user wants to see what skills/plugins are available, or asks "what can you do", "what skills do I have", etc.
---

# Task

Scan all installed skills and plugins, then display a formatted catalog to the user.

# Method: Single-Pass Scan

Run ONE bash command that scans everything at once. This avoids 15+ sequential tool calls and completes in seconds.

## Step 1: Run the scan script

Execute this bash command. It scans all marketplaces, plugins, and custom commands in a single pass and outputs structured text:

```bash
MARKET_DIR="$HOME/.claude/plugins/marketplaces"
SETTINGS="$HOME/.claude/settings.json"
PROJ_DIR="$(pwd)/.claude/commands"
SEPARATOR="|||"

extract_frontmatter() {
  local file="$1" field="$2"
  sed -n '/^---$/,/^---$/{ /^'"$field"':/{ s/^'"$field"': *//; s/^"//; s/"$//; p; } }' "$file" 2>/dev/null | head -1
}

extract_json_field() {
  local file="$1" field="$2"
  sed -n 's/.*"'"$field"'": *"\([^"]*\)".*/\1/p' "$file" 2>/dev/null | head -1
}

# --- Skills from marketplaces ---
skills=""
seen_skills=$(mktemp)
trap "rm -f '$seen_skills'" EXIT

add_skill() {
  local sname="$1" sfile="$2" source="$3"
  grep -qxF "$sname" "$seen_skills" 2>/dev/null && return
  local desc=$(extract_frontmatter "$sfile" "description")
  [ -z "$desc" ] && desc="(no description)"
  echo "$sname" >> "$seen_skills"
  skills="${skills}${sname}${SEPARATOR}/${sname}${SEPARATOR}${desc}${SEPARATOR}${source}\n"
}

for market in "$MARKET_DIR"/*/; do
  mname=$(basename "$market")
  # Pattern 1: marketplace/skills/skill-name/ (anthropics-skills style)
  if [ -d "$market/skills" ]; then
    for skill_dir in "$market/skills"/*/; do
      [ -f "$skill_dir/SKILL.md" ] || continue
      add_skill "$(basename "$skill_dir")" "$skill_dir/SKILL.md" "$mname"
    done
  fi
  # Pattern 2: marketplace/plugin-name/skills/skill-name/ (pm-skills style)
  for plugin_dir in "$market"*/; do
    [ -d "$plugin_dir/skills" ] || continue
    for skill_dir in "$plugin_dir/skills"/*/; do
      [ -f "$skill_dir/SKILL.md" ] || continue
      add_skill "$(basename "$skill_dir")" "$skill_dir/SKILL.md" "$(basename "$(dirname "$(dirname "$skill_dir")")")"
    done
  done
  # Pattern 3: marketplace/plugins/plugin-name/skills/skill-name/ (claude-plugins-official style)
  if [ -d "$market/plugins" ]; then
    for plugin_dir in "$market/plugins"/*/; do
      [ -d "$plugin_dir/skills" ] || continue
      for skill_dir in "$plugin_dir/skills"/*/; do
        [ -f "$skill_dir/SKILL.md" ] || continue
        add_skill "$(basename "$skill_dir")" "$skill_dir/SKILL.md" "$mname"
      done
    done
  fi
done

# --- Plugin commands from plugins/*/commands/ ---
plugin_cmds=""
for market in "$MARKET_DIR"/*/; do
  [ -d "$market/plugins" ] || continue
  mname=$(basename "$market")
  for plugin_dir in "$market/plugins"/*/; do
    pname=$(basename "$plugin_dir")
    if [ -d "$plugin_dir/commands" ]; then
      for cmd_file in "$plugin_dir/commands"/*.md; do
        [ -f "$cmd_file" ] || continue
        cname=$(basename "$cmd_file" .md)
        desc=$(extract_frontmatter "$cmd_file" "description")
        [ -z "$desc" ] && desc="(no description)"
        plugin_cmds="${plugin_cmds}${pname}${SEPARATOR}/${cname}${SEPARATOR}${desc}${SEPARATOR}${mname}\n"
      done
    fi
  done
done

# --- Background plugins (no commands) ---
# Pre-load descriptions from marketplace.json for plugins without plugin.json
marketplace_descs=""
for market in "$MARKET_DIR"/*/; do
  mf="$market/.claude-plugin/marketplace.json"
  [ -f "$mf" ] || continue
  descs=$(awk -F'"' '
    /"name":/ { for(i=1;i<=NF;i++) if($i=="name") { gsub(/[ ,]/,"",$(i+2)); n=$(i+2); break } }
    /"description":/ { for(i=1;i<=NF;i++) if($i=="description") { d=$(i+2); if(n && d) print n "|||" substr(d,1,150); n=""; d=""; break } }
  ' "$mf" 2>/dev/null)
  marketplace_descs="${marketplace_descs}${descs}\n"
done

bg_plugins=""
for market in "$MARKET_DIR"/*/; do
  [ -d "$market/plugins" ] || continue
  mname=$(basename "$market")
  for plugin_dir in "$market/plugins"/*/; do
    pname=$(basename "$plugin_dir")
    if [ ! -d "$plugin_dir/commands" ]; then
      # Skip plugins that have skills (already listed as skills)
      [ -d "$plugin_dir/skills" ] && continue
      desc=""
      # Try plugin.json first
      if [ -f "$plugin_dir/.claude-plugin/plugin.json" ]; then
        desc=$(extract_json_field "$plugin_dir/.claude-plugin/plugin.json" "description")
      fi
      # Fallback: look up in marketplace.json
      if [ -z "$desc" ] && [ -n "$marketplace_descs" ]; then
        desc=$(echo -e "$marketplace_descs" | sed -n "s/^${pname}|||//p" | head -1)
      fi
      [ -z "$desc" ] && desc="(no description)"
      bg_plugins="${bg_plugins}${pname}${SEPARATOR}${desc}${SEPARATOR}${mname}\n"
    fi
  done
done

# --- Custom commands ---
custom=""
if [ -d "$HOME/.claude/commands" ]; then
  for f in "$HOME/.claude/commands"/*.md; do
    [ -f "$f" ] || continue
    cname=$(basename "$f" .md)
    desc=$(extract_frontmatter "$f" "description")
    [ -z "$desc" ] && desc="(no description)"
    custom="${custom}${cname}${SEPARATOR}/${cname}${SEPARATOR}${desc}${SEPARATOR}user-global\n"
  done
fi
if [ -d "$PROJ_DIR" ]; then
  for f in "$PROJ_DIR"/*.md; do
    [ -f "$f" ] || continue
    cname=$(basename "$f" .md)
    desc=$(extract_frontmatter "$f" "description")
    [ -z "$desc" ] && desc="(no description)"
    custom="${custom}${cname}${SEPARATOR}/${cname}${SEPARATOR}${desc}${SEPARATOR}project\n"
  done
fi

# --- Output ---
echo "===SKILLS==="
echo -e "$skills"
echo "===PLUGIN_COMMANDS==="
echo -e "$plugin_cmds"
echo "===BG_PLUGINS==="
echo -e "$bg_plugins"
echo "===CUSTOM==="
echo -e "$custom"
```

## Step 2: Parse and display

The script outputs sections separated by `===SECTION_NAME===` markers. Each line within a section uses `|||` as the field separator.

Parse the output and display a formatted catalog with these sections:

### Output Format

```
## Installed Skills & Plugins Catalog

### Skills (from marketplaces)

| # | Name | Slash Command | Description | Source |
|---|------|--------------|-------------|--------|
| 1 | pdf | /pdf | Use this skill whenever... | anthropics-skills |
| ... | ... | ... | ... | ... |

### Plugin Commands

| # | Plugin | Slash Command | Description | Marketplace |
|---|--------|--------------|-------------|-------------|
| 1 | commit-commands | /commit | Create a git commit | claude-plugins-official |
| ... | ... | ... | ... | ... |

### Background Plugins (no commands)

| # | Plugin | Description | Marketplace |
|---|--------|-------------|-------------|
| 1 | security-guidance | Security review for... | claude-plugins-official |
| ... | ... | ... | ... |

### Custom Commands

| # | Name | Slash Command | Description | Location |
|---|------|--------------|-------------|----------|
| 1 | my-skills | /my-skills | List all installed skills... | user-global |
| ... | ... | ... | ... | ... |

### Summary

| Category | Count |
|----------|-------|
| Marketplace skills | X |
| Plugin commands | X |
| Background plugins | X |
| Custom commands | X |
| **Total slash commands** | **X** |
| **Total enabled plugins** | **X** |
```

# Important Notes

- Do NOT modify any files. This is a read-only operation.
- Show ALL results, not just a summary.
- If the list is very long, still show everything — the user wants a complete picture.
- The entire scan should complete in ONE bash call. Do not make additional file reads.
- Truncate long descriptions to ~100 characters in the table for readability.
