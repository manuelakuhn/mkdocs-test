# Data organization

## Data structure on Medusa

```bash
Medusa
└── home
    ├── data
    │   ├── medusa_user_project
    │   │   └── scratch
    │   └── other_medusa_user_project
    │       └── scratch
    └── medusa_user_home
        └── scratch

```

## Data structure explanation

All data in the `/home` directory is available across the entire cluster.

`/home/<user_name>`</b>  
This directory is for all of your personal files.</b>  
`/home/data/<project_name>`</b>  
This directory is for data shared across the group/project.</b>  
`/home/<user_name>/scratch` or `/home/data/<project_name>/scratch`</b>  
This directory is not backed-up and should be used to store interim results which can be easily regenerated. Storing data here helps relieve the burden on backups.</b>  
`/home/data/archive/<project_name>`</b>  
Read-only and heavily compressed (via cool transparent compression mojo), this directory stores data for projects which are completed.

## How to transfer files from/to Medusa

`rsync` is the most efficient tool you can use to transfer data from and to Medusa. Right now you can use it on Linux, macOS and Windows

**From your computer to Cecile:**

```bash
$ rsync -rltoDvh --progress </my_computer/some_data> <user@cecile.ovgu.de:~/target_directory/>

```

**From Cecile to your computer:**
Also in this case the following command should be typed on your machine.

```bash
$ rsync -rltoDvh --progress <~/some_directory_on_cecile/some_data> <~/my_computer/>
```

**WinSCP or Cyberduck**

If you prefer using a GUI, WinSCP (Windows) and Cyberduck (macOS and Windows) are decent SFTP clients.

To connect to Medusa: install and launch your client. Enter the information for host (sftp://medusa.ovgu.de), user, and password. Click connect. Once connected, you can drag-and-drop files to transfer.

## Backups

All data located under the `/home` directory are snapshot at 05:00 every day — *except* for any data located in the `/home/<user_name>/scratch` or `/home/data/<project_name>/scratch` folders.
Snapshots are then transferred to a dedicated backup server.

The backup retention policy is:

- **daily backups**

  * executed daily at 05:00
  * kept for 2 weeks

- **weekly backups**

  * executed on Mondays at 05:10
  * kept for 6 weeks

- **monthly backups**

  * executed on the 1st of every month at 05:20
  * kept for 1 year