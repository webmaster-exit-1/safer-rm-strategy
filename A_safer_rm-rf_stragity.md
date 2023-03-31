# A safer `rm -rf` strategy

Create a trash directory:

```bash
mkdir ~/.trash
```

Add the custom `safe_rm` **function** to your shell configuration file (e.g., `~/.bashrc` or `~/.zshrc`):

```bash
safe_rm() {
  trash_dir="${HOME}/.trash"

  for item in "$@"; do
    timestamp=$(date +%Y%m%d%H%M%S)
    filename=$(basename "$item")
    target="${trash_dir}/${timestamp}_${filename}"
    mv "$item" "$target"
    echo "Moved $item to $target"
  done
}

alias rm='safe_rm'
```

This function moves files or directories to the trash folder with a timestamp appended to their names to avoid conflicts.

Reload your shell configuration:

```bash
source ~/.bashrc
```

or

```bash
source ~/.zshrc
```

Create a script to delete old files from the trash directory:
Create a file called `clean_trash.sh`:

```bash
touch ~/clean_trash.sh
chmod +x ~/clean_trash.sh
```

Open the file with your favorite text editor and add the following content:

```bash
#!/bin/bash

trash_dir="${HOME}/.trash"
max_age=7 # Age in days before files are deleted

find "$trash_dir" -type f -mtime +$max_age -exec rm -f {} \;
find "$trash_dir" -type d -empty -mtime +$max_age -exec rmdir {} \;
```

This script will delete files older than the specified max_age (in days) and remove empty directories.

Set up a cron job to run the `clean_trash.sh` script daily:
Open your crontab with `crontab -e` and add the following line:

```ruby
0 0 * * * /bin/bash /path/to/your/clean_trash.sh
```

This will run the `clean_trash.sh` script every day at midnight.

Now, when you use the `rm` command, files and directories will be moved to the trash folder instead of being permanently deleted. The cron job will automatically clean up items older than the specified max_age.

Create a script called `notify_trash.sh`:

```bash
touch ~/notify_trash.sh
chmod +x ~/notify_trash.sh
```

Open the script with your favorite text editor and add the following content:

```bash
#!/bin/bash

trash_dir="${HOME}/.trash"
remaining_hours=48
count=$(find "$trash_dir" -mindepth 1 | wc -l)

if [ "$count" -gt 0 ]; then
  if command -v notify-send >/dev/null; then
    notify-send "Trash Reminder" "You have ${count} item(s) in your trash folder. You have ${remaining_hours} hours to restore them before they are permanently deleted."
  else
    echo "Trash Reminder: You have ${count} item(s) in your trash folder. You have ${remaining_hours} hours to restore them before they are permanently deleted."
  fi
fi
```

This script checks if there are files or folders in the `.trash` directory. If there are, it sends a notification using the `notify-send` command, which is available on most Linux desktop environments. If `notify-send` is not available, it will print the reminder message to the terminal.

Set up a cron job to run the `notify_trash.sh` script every 2 hours:
Open your crontab with crontab -e and add the following line:

```ruby
0 */2 * * * /bin/bash /path/to/your/notify_trash.sh
```
