# Neuro-symbolic LLOD

## Minutes

Minutes are stored in a rolling google document: https://docs.google.com/document/d/1Q0TZZDNZ7vSrENVkKSxy-ym8mJnS5nE2Ozz3TWTkrdM/edit
For sustainability and consistency reasons, we would keep a backup of the minutes within the git repository as well, using the `sync-gdrive.sh`

In particular, this pertains to the folder
- `minutes/`
    
For updating the local mirror, run 

`$> bash -e ./sync-gdrive.sh`

See notes in `sync-gdrive.sh` for how to configure the rclone command.

To facilitate searchability, `sync-gdrive.sh` will also produce a text export of the GDrive minutes under `minutes_txt/`.
