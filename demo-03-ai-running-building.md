# Demo 03 — Running WP-CLI with AI + Building Custom Commands

**CLAUDE.md or .github/copilot-instructions.md must contain:**

```markdown
# WP-CLI — Context

## WordPress setup

- Path: /Users/$(whoami)/Sites/wplab
- URL: https://wplab.test
- WP-CLI is installed and available as `wp`
- Always use --path=/Users/$(whoami)/Sites/wplab with every wp command

## How to behave

- Show the exact WP-CLI command before running it
- Run read-only commands freely
- Ask for confirmation before any command that modifies data
- Explain results in plain English after each command
```

## PART A — Running WP-CLI With Claude Code

### What This Is

Claude Code runs inside VS Code. It reads CLAUDE.md automatically, executes real `wp` commands against `~/Sites/wplab`, and returns the results in plain English.

### Read-Only Prompts

Type each prompt in the Claude Code panel. Pause after each to narrate what Claude did.

**Prompt 1:**

```
What WordPress version is running and are there any updates available?
```

**Prompt 2:**

```
List all active plugins with their versions
```

**Prompt 3:**

```
How many published posts, pages, and users does this site have?
```

**Prompt 4:**

```
Are there any inactive plugins that could be cleaned up?
```

### Write Operation — The Permission Prompt

**Prompt 5:**

```
Clear all transients from the database
```

Claude should respond with something like:

```
I'll run:
wp --path=~/Sites/wplab transient delete --all

This deletes all transients from the database. Shall I proceed?
```

### Chained Diagnostic Prompt

**Prompt 6:**

```
Quick health check — are any plugins out of date, what cron events are due soon, and is WP_DEBUG currently on?
```

## PART B — Building Custom WP-CLI Commands With AI

### What This Is

Starting from a plain English description, AI writes a complete PHP WP-CLI command. The demo goes from description → PHP file → activated plugin → working terminal command.

### Custom command example

```php
<?php
if ( ! defined( 'WP_CLI' ) || ! WP_CLI ) {
    return;
}

class Mysite_Command extends WP_CLI_Command {
    public function hello( $args, $assoc_args ) {
        WP_CLI::success( 'Hello from my custom command!' );
    }
}

WP_CLI::add_command( 'mysite', 'Mysite_Command' );
```

### Scaffold the Command with AI

**Type this prompt:**

```
Create a custom WP-CLI command for WordPress.

Command group: "mysite"
Subcommand: "stats"

It should output:
- Total number of published posts
- Total number of registered users
- Current WordPress version
- Active theme name

Use WP_CLI::log() for each stat and WP_CLI::success() at the end.
Add a comment explaining each part for a beginner audience.
Include the WP_CLI::add_command() registration.
Add a WP-CLI docblock with DESCRIPTION, OPTIONS and EXAMPLES sections.
```

### Install and Run It

Copy the generated code. In VS Code, create the mu-plugin file:

```
~/Sites/wplab/wp-content/mu-plugins/mysite-commands.php
```

The file needs a plugin header at the top:

```php
<?php
/**
 * Plugin Name: My Site WP-CLI Commands
 * Description: Custom WP-CLI commands
 * Version: 1.0
 */
```

Then paste the generated code below the header.

In the VS Code integrated terminal:

```bash

# Run the new command
wp --path=~/Sites/wplab mysite stats

# Check the auto-generated help text
wp --path=~/Sites/wplab help mysite stats
```

### Iterate — Adding a Flag

**Type this prompt:**

```
Extend the stats() method to accept a --format flag supporting table, json, and csv. Default to table.
Use WP_CLI\Utils\format_items() for the output.
Show only the updated stats() method.
```

Copy the updated method, replace the old one in VS Code, then test:

```bash
wp --path=~/Sites/wplab mysite stats
wp --path=~/Sites/wplab mysite stats --format=json
wp --path=~/Sites/wplab mysite stats --format=csv
```
