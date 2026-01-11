========================================
Tape backup for homefolders and projects
========================================

For homefolders snapshots are taken weekly and monthly, these snapshots are being written to tape in irregular intervals. Previous backup overview:

+---------------------+---------------------------------------------+---------------------------------------+
| Date                | What                                        | Notes                                 |
+=====================+=============================================+=======================================+
| 08-10-2025          | Full backups of project folders             | Being done by Ameer, not confirmed yet|
+---------------------+---------------------------------------------+---------------------------------------+
| 08-10-2025          | Full backup of homefolders                  | Confirmed written to tape             |
+---------------------+---------------------------------------------+---------------------------------------+
| 04-10-2024          | Snapshots of homefolders and projects < 1TB | Confirmed and written to tape         |
+---------------------+---------------------------------------------+---------------------------------------+

In October 2025, a full backup of all home and project folders is / will be done by Ameer. In the future this will be the basis from which we can make incremental snapshot backups.

The process of creating tape backups is as follows:

1. Generate snapshots for the folders to be backed up.

2. Ask Ameer to which location the data should be written. Ameer will typically
   make a mounted folder available on the system.

3. If written to the folder mention to Ameer and he will write the data to tape.

4. Verify with Ameer that the data has been written to tape.


Contact person and responsible for doing tape backups is Marek Oerlemans, reachable via nki-ai slack, teams or email.
