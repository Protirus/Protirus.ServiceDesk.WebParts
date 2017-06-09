# Protirus.ServiceDesk.WebParts
Symantec ServiceDesk WebParts

See on [Symantec Connect](https://www.symantec.com/connect/downloads/servicedesk-webpart-assignments-and-audit-info).

Version	SD 7.6

This download ([Protirus.ServiceDesk.WebParts.dll](https://github.com/Protirus/Protirus.ServiceDesk.WebParts/releases/latest)) is a couple of custom Web Parts to show 'ServiceDesk Assignments' and the 'Auditing Incident Owner and Service Queue Changes'.

![WebParts - SD Assignments and Queue Info](https://github.com/Protirus/Protirus.ServiceDesk.WebParts/blob/master/WebParts%20-%20SD%20Assignments%20and%20Queue%20Info.png)

If you haven't see the following Article/Video I would advise doing so first.

ServiceDesk 7.5 Track Assignments via Process Type Actions
https://www-secure.symantec.com/connect/articles/servicedesk-75-track-assignments-process-type-actions

Send Incident to Workflow - Auditing Incident Owner and Service Queue Changes
https://www-secure.symantec.com/connect/videos/send-incident-workflow-auditing-incident-owner-and-service-queue-changes

The idea is to save having to add Process Type Actions and an extra page to open and view, you can have all the information at your disposal on the Process View Page.

The Track Assignments you can use with only adding the below Stored Procedure to your DB but for the Auditing you will need to follow the video to add in the necessary tables via the Web App project.

As you can see in the Track Assignments article there is text file "track_all_assignments.txt" which we need to convert to a Stored Procedure, just add in the SessionID parameter and save as "sp_SD_TrackAssignments"

[sp_SD_TrackAssignments](https://github.com/Protirus/Protirus.ServiceDesk.WebParts/blob/master/sp_SD_TrackAssignments.txt)

Next we want a couple of SPs for the Audit info:

[sp_SD_AuditProcessInfo](https://github.com/Protirus/Protirus.ServiceDesk.WebParts/blob/master/sp_SD_AuditProcessInfo.txt)
```SQL
SELECT 
	[audit_process_info_id]
	,[incident_session_id]
	,[assigned_to_queue_name]
	,[assigned_on_date]
	,CASE 
		WHEN [assigned_away_date] = '1753-01-01 00:00:00.000'
			THEN
				NULL
			ELSE
				[assigned_away_date]
	END 
		AS [assigned_away_date]
	,CASE 
		WHEN [assigned_away_date] = '1753-01-01 00:00:00.000'
			THEN
				NULL 
			ELSE
				DATEDIFF(mi, [assigned_on_date], [assigned_away_date]) 
	END 
		AS [time_in_queue]
	,[current_assignment]
FROM 
	[audit_process_info]
WHERE 
	[incident_session_id] = @sessionid
ORDER BY
	[assigned_on_date] DESC
```

[sp_SD_OwnerAuditInfo](https://github.com/Protirus/Protirus.ServiceDesk.WebParts/blob/master/sp_SD_OwnerAuditInfo.txt)

```SQL
SELECT 
	[owner_audit_info_id]
	,[is_current_owner]
	,[owner_email]
	,[ownership_taken_on]
	,CASE 
		WHEN [ownership_relinquished] = '1753-01-01 00:00:00.000'
			THEN
				NULL
			ELSE
				[ownership_relinquished]
	END 
		AS [ownership_relinquished]
	,CASE 
		WHEN [ownership_relinquished] = '1753-01-01 00:00:00.000'
			THEN
				NULL 
			ELSE
				DATEDIFF(mi, [ownership_taken_on], [ownership_relinquished]) 
	END 
		AS [ownership_time]
	,[incident_session_id]
FROM 
	[owner_audit_info]
WHERE
	[incident_session_id] = @sessionid
ORDER BY
	[ownership_taken_on] DESC
```

Just update the first line to match the name of your Process Manager Instance and rename from ".txt" to ".sql" and run in your SSMS.

```USE [ProcessManager]```

Now we need to upload the WebPart

Login to Process Manager

Admin -> Portal -> Plugin Upload 

Choose WebPart from the "Plugin Type" and browse for the dll then click Upload.

This can take some time.

It will be saved to "[workflow dir]:\\ProcessManager\bin"

You may need an IIS Reset or WF Services restart.

Go to Admin -> Portal -> Web Parts Catalog

Click on the + button and then give it a new Category name and search for each and add individually

- SDIncidentServiceQueueHistory
- SDTrackAssignmentsWebPart

Now these are available to add to the Process View Page.

Go to the Page in Admin -> Portal -> Manage Pages

Find your page and "Go To Page"

Site Actions | ModifyPage

Site Actions | Edit Page

Site Actions | Add Web Part

Now add the new web parts from the new Category you created and give them titles.

Here are a couple of examples depending on the state of the ticket:

![WebParts - SD Assignments and Queue Info](https://github.com/Protirus/Protirus.ServiceDesk.WebParts/blob/master/WebParts%20-%20SD%20Assignments%20and%20Queue%20Info%20(All).png)

---

**Old Links**

How to add a custom web part to a ServiceDesk/Workflow page in ServiceDesk/ Workflow 7.1 SP2 ?

https://support.symantec.com/en_US/article.TECH192190.html

Custom Data Types, Incident Forms, and the Process View Page – Putting them All Together

https://www-secure.symantec.com/connect/articles/custom-data-types-incident-forms-and-process-view-page-putting-them-all-together
