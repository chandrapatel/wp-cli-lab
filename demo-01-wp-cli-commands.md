# Demo 01 — WP-CLI Commands

## DEMO: Environment Check

**Purpose:** Confirm WP-CLI is working and show what environment info looks like.

```bash
wp --info
wp --version
```

**Expected output from `wp --info`:**

```bash
OS:     Darwin 24.x.x
Shell:  /bin/zsh
PHP binary:     /usr/local/bin/php
PHP version:    8.x.x
WP-CLI version: 2.x.x
```

## DEMO: Built-in Help

**Purpose:** Show that every command has documentation built in — no browser needed.

```bash
wp help
wp help plugin
wp help plugin install
```

## DEMO: wp core

**Purpose:** Check and manage WordPress core.

```bash
cd ~/Sites/wplab

wp core version
wp core check-update
```

## DEMO: wp plugin

**Purpose:** The most commonly used command group — install, manage, update plugins.

```bash
# List all plugins with status
wp plugin list

# Filter by status
wp plugin list --status=active
wp plugin list --status=inactive

# Different output formats
wp plugin list --format=json
wp plugin list --format=csv

# Install a plugin
wp plugin install akismet

# Install and activate in one step
wp plugin install akismet --activate

# Update a specific plugin
wp plugin update contact-form-7

# Update ALL plugins at once
wp plugin update --all

# Deactivate then delete
wp plugin deactivate akismet
wp plugin delete akismet
```

## DEMO: wp theme

**Purpose:** Theme management follows the exact same pattern as plugins.

```bash
wp theme list
wp theme install twentytwentyfour
wp theme activate twentytwentyfour
wp theme update --all
wp theme list --status=active
```

## DEMO: wp user

**Purpose:** User management including the locked-out scenario.

```bash
# List users
wp user list
wp user list --fields=ID,user_login,user_email,roles

# Get a specific user
wp user get 1
wp user get editor1@local.dev --by=email

# Create a new user
wp user create testuser testuser@local.dev --role=editor

# Reset password without sending notification email
wp user update 1 --user_pass=newpassword --skip-email

# Generate test users
wp user generate --count=5 --role=subscriber

# List again to see the generated users
wp user list --fields=ID,user_login,roles
```

## DEMO: wp post

**Purpose:** Content management from the terminal.

```bash
# List posts
wp post list
wp post list --post_type=page --post_status=publish
wp post list --fields=ID,post_title,post_status

# Create a post
wp post create \
  --post_title="Demo Post" \
  --post_content="Created from the terminal." \
  --post_status=publish

# Get the ID from the output, then update it
wp post update <ID> --post_title="Updated Demo Post"

# Generate test content
wp post generate --count=10 --post_type=post

# Delete (sends to trash by default)
wp post delete <ID>

# Permanent delete
wp post delete <ID> --force
```

## DEMO: wp db

**Purpose:** Database backup, import, and the critical search-replace command.

```bash
# Backup — do this before anything destructive
wp db export backup-$(date +%Y%m%d).sql

# Verify the file was created
ls -lh *.sql

# Run a SQL query without opening phpMyAdmin
wp db query "SELECT ID, post_title FROM wp_posts WHERE post_status='publish' LIMIT 5"

# Search and replace — ALWAYS dry-run first
wp search-replace 'https://wplab.test' 'https://mysite.com' --dry-run

# Read the output, then apply
wp search-replace 'https://wplab.test' 'https://mysite.com'

# Restore from backup
wp db import backup-$(date +%Y%m%d).sql
```

## DEMO: wp option

**Purpose:** Read and write values in the wp_options table.

```bash
# Read common options
wp option get siteurl
wp option get blogname
wp option get active_plugins

# Update an option
wp option update blogdescription "A WordPress site managed from the terminal"

# Check search indexing status
wp option get blog_public

# Disable search indexing (useful on staging/local)
wp option update blog_public 0
```

## DEMO: wp config

**Purpose:** Read and write wp-config.php values without opening a text editor.

```bash
# Read config values
wp config get WP_DEBUG
wp config get DB_NAME

# Enable debug mode
wp config set WP_DEBUG true --raw
wp config set WP_DEBUG_LOG true --raw

# Verify
wp config get WP_DEBUG

# Turn it back off
wp config set WP_DEBUG false --raw

# See all constants defined in wp-config.php
wp config list
```

## DEMO: wp eval

**Purpose:** Run PHP code inside the WordPress context — useful for debugging.

```bash
# Test a WordPress function directly
wp eval 'echo get_bloginfo("name") . "\n";'

# Check what an option actually contains
wp eval 'var_dump(get_option("active_plugins"));'

# Generate a secure password
wp eval 'echo wp_generate_password(16, false);'

# Test a filter
wp eval 'echo apply_filters("the_title", "Test Title", 1) . "\n";'
```

## DEMO: Workflow — Fresh WordPress Site in 60 Seconds

**Purpose:** Show the full new site setup workflow from empty directory to working WordPress.

```bash
# Create a fresh directory
mkdir ~/Desktop/demosite && cd ~/Desktop/demosite

# Download WordPress
wp core download

# Create wp-config.php
wp config create \
  --dbname=demosite \
  --dbuser=root \
  --dbpass= \
  --dbhost=127.0.0.1

# Create the database
wp db create

# Install WordPress
wp core install \
  --url=http://demosite.test \
  --title="Demo Site" \
  --admin_user=admin \
  --admin_password=admin \
  --admin_email=admin@example.com

# Verify
wp core version
wp option get siteurl

# Clean up defaults
wp plugin delete hello
wp post delete 1 2 --force
```

## DEMO: Workflow — Local Development Sync

**Purpose:** Show the complete workflow for safely syncing a production database to a local environment.

**Before running:** Copy `prod-dump.sql` to `~/Sites/wplab/`
```bash
cp ~/Desktop/prod-dump.sql ~/Sites/wplab/prod-dump.sql
```

Walk through each command individually and explain as you go.

```bash
cd ~/Sites/wplab

# Show current state before import
wp user list --fields=ID,user_login,user_email
wp option get siteurl

# STEP 1 — Import the production database
wp db import prod-dump.sql

# STEP 2 — Preview URL replacement first
wp search-replace 'https://mysite.com' 'https://wplab.test' --dry-run

# STEP 2 — Apply URL replacement
wp search-replace 'https://mysite.com' 'https://wplab.test'

# STEP 3 — Show user data before anonymisation
wp user list --fields=ID,user_login,user_email,display_name

# STEP 3 — Anonymise all user PII
wp user list --field=ID | while read ID; do
  wp user update $ID \
    --user_email="user_${ID}@local.dev" \
    --display_name="Test User ${ID}" \
    --user_pass="localpass123" \
    --skip-email
done

# STEP 3 — Show user data after anonymisation
wp user list --fields=ID,user_login,user_email,display_name

# STEP 4 — Deactivate production-only plugins
wp plugin deactivate wp-rocket sendgrid-email-delivery-simplified wordfence 2>/dev/null || true

# STEP 5 — Disable search engine indexing
wp option update blog_public 0
wp option get blog_public

# STEP 6 — Remove production API keys
wp option delete sendgrid_api_key
wp option delete mailchimp_api_key

# STEP 7 — Enable local debug mode
wp config set WP_DEBUG true --raw
wp config set WP_DEBUG_LOG true --raw

# STEP 8 — Flush everything
wp cache flush
wp rewrite flush

echo "Sync complete"
```
