# Shared folders
If you need to give other users write access to a file/folder that you own, you need to set the group ownership of the folder to the `bio_server_users@bio.aau.dk` group and set the [setGID](https://www.geeksforgeeks.org/setuid-setgid-and-sticky-bits-in-linux-file-permissions/) bit on folders (to ensure child files/folders will inherit the ownership of a parent folder), see the example below. This will give **everyone** with access to the BioCloud servers full control of the files. If you only want a specific group of people to have write access, there is only one way to do that, which is to contact the university IT services to create an email address group for the specific users, and then follow the same steps below, but instead use the new email of that group.

**Never** use `chmod 777`. It's a major security weakness that can allow anyone (even users outside of the university authentication system, human or not) to do anything they want with a file/folder and potentially place hidden payload there. Folders will be scanned regularly for insecure permissions and corrected without notice. Only `owner` and `group` should have the execute bit set, never `other`. See [permissions in Linux](https://www.geeksforgeeks.org/permissions-in-linux/) to learn about file and folder permissions on Linux systems.

```
folder="some_folder/"

# create the folder if it doesn't exist already
mkdir -p "$folder"

# set group ownership
chown -R :bio_server_users@bio.aau.dk "$folder"

# If there are already files with weak permissions, correct them with:
chmod -R o-x "$folder"

# Ensure group can edit files/folders:
find "$folder" -type f -exec chmod 664 {} \;
find "$folder" -type d -exec chmod 775 {} \;

# set the setGID sticky bit to ensure new files and folders inherit group ownership
chmod 2775 "$folder"
```
