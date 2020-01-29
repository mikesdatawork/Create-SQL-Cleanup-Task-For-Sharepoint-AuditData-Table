![MIKES DATA WORK GIT REPO](https://raw.githubusercontent.com/mikesdatawork/images/master/git_mikes_data_work_banner_01.png "Mikes Data Work")        

# Create SQL Cleanup Task For Sharepoint AuditData Table
**Post Date: April 3, 2018**





## Contents    
- [About Process](##About-Process)  
- [SQL Logic](#SQL-Logic)  
- [Build Info](#Build-Info)  
- [Author](#Author)  
- [License](#License)       

## About-Process



![SQL Cleanup Task]( https://mikesdatawork.files.wordpress.com/2018/04/image001.png "Cleanup Sharepoint AuditData Table")
 
<p>Some Sharepoint WSS_Content databases will have a substantially large table called [AuditData]. This is a basic record of access times for the Sharepoint environment. Most of this data is benign and rarely looked at, and with alittle bit of Sharepoint managmenet these records can be regularly maintained, but often with most environments; the little things go unseen. It's only after a long period of time that these logs can get out of hand. In this case millions of rows. Fortunately there aren't any foreign keys on the table (as of the writing on this post) so maintenance can be pretty basic.
In this case I created an SQL Cleanup Task to remove some of the old [AuditData] entries. This keeps the most recent 91 days. Why 91 days? Cause it will keep all previous 90 days (plus tomorrow) this will guarantee any new rows added while the cleanup task is running will be saved.
Note: This is a batch process. Presently it's set to only delete 10 rows at a time. This of course can be adjusted, but I have it this way so the task it's self is an extremely minimal operation.</p>   


## SQL-Logic
```SQL
use My_WSS_CONTENT_DATABASE;
set nocount on
 
declare @start_date datetime
declare @end_date   datetime
set @start_date = (select dateadd(d, +1, getdate()))
set @end_date   = (select dateadd(d, -90, getdate()))
 
declare @cleanup_task   varchar(max)
set @cleanup_task   =
'
set nocount on
while exists ( select * from [AuditData] where [occurred] not between ''' + CONVERT(nvarchar(24), @start_date, 121) + ''' and ''' + CONVERT(nvarchar(24), @end_date, 121) + ''')
    begin
        begin tran spauditmaint
        delete top (10) from [AuditData] where [occurred] not between ''' + CONVERT(nvarchar(24), @start_date, 121) + ''' and ''' + CONVERT(nvarchar(24), @end_date, 121) + '''
        commit tran spauditmaint
        checkpoint;
        print ''10 rows deleted''
    end
'
exec (@cleanup_task)
```


If you're looking to hit ALL content databases you can try this one.
Each content database could easily have millions of rows under it's AuditData tables so doing batch deletes of 500 rows at a time is good as the system can better digest the deletes. Each delete has a commit, and checkpoint so it's easier to manage, but the process does take a while. Days perhaps. Because of this I set the 'between' days to look between 90 days past, and 7 days into the future. This will give your process time to complete among a variety of databases. Again; be sure to run periodic shrink operations, and it would be helpful to have an uptick in your transaction log backups, or better yet; setting your database to SIMPLE recovery while this process is carried out.  


## SQL-Logic
```SQL
use master;
set nocount on
 
declare @clean_process  varchar(max)
set @clean_process  = ''
select  @clean_process  = @clean_process +
'use [' + [name] + '];
set nocount on
 
declare @start_date_' + cast([database_id] as varchar) + '  datetime
declare @end_date_' + cast([database_id] as varchar) + '    datetime
set @start_date_' + cast([database_id] as varchar) + '  = (select dateadd(d, +7,    getdate()))
set @end_date_' + cast([database_id] as varchar) + '    = (select dateadd(d, -90,   getdate()))
 
declare @cleanup_task_' + cast([database_id] as varchar) + '    varchar(max)
set @cleanup_task_' + cast([database_id] as varchar) + '    =
''
set nocount on
while exists ( select * from [AuditData] where [occurred] not between '''''' + CONVERT(nvarchar(24), @start_date_' + cast([database_id] as varchar) + ', 121) + '''''' and '''''' + CONVERT(nvarchar(24), @end_date_' + cast([database_id] as varchar) + ', 121) + '''''')
    begin
        begin tran spauditmaint
        delete top (500) from [AuditData] where [occurred] not between '''''' + CONVERT(nvarchar(24), @start_date_' + cast([database_id] as varchar) + ', 121) + '''''' and '''''' + CONVERT(nvarchar(24), @end_date_' + cast([database_id] as varchar) + ', 121) + ''''''
        commit tran spauditmaint
        checkpoint;
        print ''''500 rows deleted under log table for database [' + upper([name]) + ']''''
    end
''
exec (@cleanup_task_' + cast([database_id] as varchar) + ')
' + char(10) + char(10)
from    sys.databases where [name] like '%content%'
order by [name] desc
 
exec (@clean_process) --for xml path (''), type
```


[![WorksEveryTime](https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg)](https://shitday.de/)

## Build-Info

| Build Quality | Build History |
|--|--|
|<table><tr><td>[![Build-Status](https://ci.appveyor.com/api/projects/status/pjxh5g91jpbh7t84?svg?style=flat-square)](#)</td></tr><tr><td>[![Coverage](https://coveralls.io/repos/github/tygerbytes/ResourceFitness/badge.svg?style=flat-square)](#)</td></tr><tr><td>[![Nuget](https://img.shields.io/nuget/v/TW.Resfit.Core.svg?style=flat-square)](#)</td></tr></table>|<table><tr><td>[![Build history](https://buildstats.info/appveyor/chart/tygerbytes/resourcefitness)](#)</td></tr></table>|

## Author

[![Gist](https://img.shields.io/badge/Gist-MikesDataWork-<COLOR>.svg)](https://gist.github.com/mikesdatawork)
[![Twitter](https://img.shields.io/badge/Twitter-MikesDataWork-<COLOR>.svg)](https://twitter.com/mikesdatawork)
[![Wordpress](https://img.shields.io/badge/Wordpress-MikesDataWork-<COLOR>.svg)](https://mikesdatawork.wordpress.com/)

     
## License
[![LicenseCCSA](https://img.shields.io/badge/License-CreativeCommonsSA-<COLOR>.svg)](https://creativecommons.org/share-your-work/licensing-types-examples/)

![Mikes Data Work](https://raw.githubusercontent.com/mikesdatawork/images/master/git_mikes_data_work_banner_02.png "Mikes Data Work")

