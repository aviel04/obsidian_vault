### Default Gitaly Setup and Repository Storage

- **Automatic Configuration:** When you install GitLab 16.4 with the Omnibus package, Gitaly is configured to run on the same server.

- **Default Storage Path:** By default, Gitaly stores all Git repositories inside a directory named `repositories` within its main data directory. The standard path for this is:

	- `/var/opt/gitlab/git-data/repositories`

    This path is managed by the `git_data_dirs` setting in your GitLab configuration file.

### ⚙️ Configuring Repository Storage Paths (`git_data_dirs`)

If you want to change where GitLab stores Git repositories (for example, to use a dedicated mounted drive for more space or better performance), you need to configure the `git_data_dirs` setting in your `/etc/gitlab/gitlab.rb` file.

- **1. Edit `gitlab.rb`:**
	```bash
	sudo vim /etc/gitlab/gitlab.rb
	```
    
- **2. Configure `git_data_dirs`:**
    
    - **Single Storage Path (Most Common for New Setups):** If you just want to change the default path for all repositories:

        ```ruby
        # Example: To store repositories in /mnt/gitlab-git-data
        git_data_dirs({
          "default" => { "path" => "/mnt/gitlab-git-data" }
        })
        ```

        - **Important Before Changing:**
            - If you have existing repositories and you change this path **after** projects have been created, you will need to manually move the existing data to the new location _before_ reconfiguring, or GitLab won't find them. For a _newly installed_ system before any projects are created, you can set this.
            - Ensure the new directory (e.g., `/mnt/gitlab-git-data`) exists and has the correct permissions for the `git` user (which GitLab uses). Omnibus GitLab typically handles permissions on reconfigure, but the parent directory structure should be there.

                ```bash
                sudo mkdir -p /mnt/gitlab-git-data
                # Permissions will be set by gitlab-ctl reconfigure, but you might need to chown the mount point itself
                # sudo chown git:git /mnt/gitlab-git-data # (More accurately, gitlab-ctl reconfigure will do this for the "repositories" subdir)
                ```

- **Multiple Storage Paths (Repository Sharding - More Advanced):** GitLab also allows you to define multiple storage paths and assign projects to them. This is useful for distributing storage load or managing different tiers of storage.

```
	# Example for multiple storage paths
	# git_data_dirs({
	#   "default" => { "path" => "/var/opt/gitlab/git-data" }, # Keep a default
	#   "storage2" => { "path" => "/mnt/fast-storage/git-data" },
	#   "storage3" => { "path" => "/mnt/large-storage/git-data" }
	# })
```

New projects can then be assigned to these storage shards via GitLab Admin settings. This is generally for larger or more complex setups.

- **3. Reconfigure GitLab:** After making changes to `gitlab.rb`, you must reconfigure GitLab for the changes to take effect:

```bash
    sudo gitlab-ctl reconfigure
```
- This will update Gitaly's configuration and ensure it uses the new paths.

### ✅ Verifying Gitaly

- **Check Gitaly Status:** You can check if Gitaly and other GitLab services are running correctly:

```bash
    sudo gitlab-ctl status
```

   - You should see Gitaly listed with a status of `run`.

- **GitLab Admin UI:** Once you can log in to GitLab, you can go to **Admin Area > Monitoring > Gitaly Servers**. This dashboard will show you information about your Gitaly server(s), their health, and the storage paths they are using.