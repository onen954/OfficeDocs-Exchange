﻿---
title: Troubleshooting OWA Health Set
TOCTitle: Troubleshooting OWA Health Set
ms:assetid: eae9dbd7-2ce6-43ce-b9a1-b114fd0c3ab4
ms:mtpsurl: https://technet.microsoft.com/en-us/library/ms.exch.scom.owa(v=EXCHG.150)
ms:contentKeyID: 49720915
ms.date: 10/08/2015
mtps_version: v=EXCHG.150
---

<div data-xmlns="http://www.w3.org/1999/xhtml">

<div class="topic" data-xmlns="http://www.w3.org/1999/xhtml" data-msxsl="urn:schemas-microsoft-com:xslt" data-cs="http://msdn.microsoft.com/en-us/">

<div data-asp="http://msdn2.microsoft.com/asp">

# Troubleshooting OWA Health Set

</div>

<div id="mainSection">

<div id="mainBody">

<span> </span>

_**Applies to:** Exchange Server 2013_

_**Topic Last Modified:** 2015-03-09_

The Outlook Web App (OWA) health set monitors the overall health of the Outlook Web App service.

If you receive an alert that specifies that Outlook Web App is unhealthy, this indicates an issue that may prevent users from accessing their mailboxes by using Outlook Web App.

<span id="EXP"></span>

<div>

## Explanation

The Outlook Web App service is monitored by using the following probes and monitors.


<table>
<colgroup>
<col style="width: 25%" />
<col style="width: 25%" />
<col style="width: 25%" />
<col style="width: 25%" />
</colgroup>
<thead>
<tr class="header">
<th>Probe</th>
<th>Health Set</th>
<th>Dependencies</th>
<th>Associated Monitors</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td><p>OwaCtpProbe</p></td>
<td><p>Outlook Web App</p></td>
<td><p>Active Directory</p>
<p>Information Store</p></td>
<td><p>OwaCtpMonitor</p></td>
</tr>
</tbody>
</table>


For more information about probes and monitors, see [Server health and performance](https://technet.microsoft.com/en-us/library/jj150551\(v=exchg.150\)).

</div>

<div>

## Common issues

This probe can fail for several reasons. The following are some of the more common reasons:

  - The Outlook Web App application pool that’s hosted on the monitored Client Access server (CAS) is not responding, or the application pool that’s hosted on the Mailbox server is not responding.

  - The CAS is experiencing networking issues, and it can’t connect to the Mailbox server or the Domain Controller.

  - The monitoring account credentials are incorrect.

  - The user’s database is not mounted, or the Information Store is inaccessible for that mailbox.

  - The Information Store is not responding.

  - The Domain Controllers are not responding.

</div>

<div>

## User Action

It's possible that the service recovered after it issued the alert. Therefore, when you receive an alert that specifies that the health set is unhealthy, first verify that the issue still exists. If the issue does exist, perform the appropriate recovery actions outlined in the following sections.

<span id="verify"></span>

<div>

## Verifying the issue still exists

1.  Identify the health set name and the server name in the alert.

2.  The message details provide information about the exact cause of the alert. In most cases, the message details provide sufficient troubleshooting information to identify the root cause. If the message details are not clear, do the following:
    
    1.  Open Exchange Management Shell, and then run the following command to retrieve the details of the health set that generated the alert:
        
            Get-ServerHealth <server name> | ?{$_.HealthSetName -eq "<health set name>"}
        
        Outlook Web App health set details about server1.contoso.com, run the following command:
        
            Get-ServerHealth server1.contoso.com | ?{$_.HealthSetName -eq "OWA"}
    
    2.  Review the command output to determine which monitor reported the error. The **AlertValue** value for the monitor that issued the alert is `Unhealthy`.
    
    3.  Rerun the associated probe for the monitor that’s in an unhealthy state. Refer to the table in the Explanation section to find the associated probe. To do this, run the following command:
        
            Invoke-MonitoringProbe <health set name>\<probe name> -Server <server name> | Format-List
        
        For example, to create an Exchange ActiveSync monitoring probe on *server1.contoso.com*, run the following command:
        
            Invoke-MonitoringProbe -Identity ActiveSync.Protocol\ActiveSyncSelfTestProbe -Server server1.contoso.com
    
    4.  In the command output, review the **Result** value of the probe. If the value is **Succeeded**, the issue was a transient error, and it no longer exists. Otherwise, refer to the recovery steps outlined in the following sections.

</div>

<span id="TestMonitors"></span>

<div>

## OwaCtpMonitor Recovery Actions

An email alert from a health set contains the following information:

  - Name of the server that sent the alert

  - Full exception trace of the last error, including diagnostic data and specific HTTP header information  
    
    **Note**   You can use the information in the full exception trace to help troubleshoot the issue. The exception generated by the probe contains a Failure Reason that describes why the probe failed. For example, the exception contains the following:
    
      - **MissingKeyword**   An expected keyword was not found in the server response. In this case, the exception contains the expected keywords.
    
      - **NameResolution**   DNS resolution is failing to resolve a given server name.
    
      - **NetworkConnection**   The Probe is receiving a network connection failure when it tries to connect to the OWA app pool on CAFE.
    
      - **UnexpectedHttpResponseCode**   Response had an unexpected HTTP code. For example, the server returned a **503** HTTP code.
    
      - **RequestTimeout**   The server took too long to respond to a client request.
    
      - **ScenarioTimeout**   The probe finished successfully, but it took more than one minute to do this. This usually indicates a system that is being overloaded.
    
      - **OwaErrorPage**   Outlook Web App returned an error page. The name of the error that caused the failure is usually available in the exception message.
    
      - **OwaMailboxErrorPage**   Outlook Web App returned an error page that contains a Mailbox Store-related error. This typically indicates that the Mailbox Store is down or that mailboxes are being dismounted.
    
    The exception trace contains an important field named **FailingComponent**. The probe tries to determine the failure, such as in the following example:
    
      - **Mailbox**   The probe can reach Outlook Web App, but it can’t connect to the Mailbox Store. In this case, the probe failed, or the mailbox access latency caused the probe to fail and generate a **ScenarioTimeout** error. When these kinds of failures occur, you should check the health of the Mailbox servers.
    
      - **Active Directory**   The probe can reach Outlook Web App, but it can’t connect to Active Directory. In this case, the probe failed or the Active Directory call latencies may have caused the probe time-out. When these types of failures occur, you should check the health of the Domain Controllers, and also check the network connections between the CA and Mailbox servers, and Domain Controllers.
    
      - **Owa**   This typically means that an error occurred inside the Outlook Web App layer. When these kinds of failures occur, you must verify the health of the Outlook Web App process on the CA and Mailbox servers, and also check the network connections.
    
    The exception also contains the most recent HTTP request and response information that was received before the probe failed. The escalation body contains the path to probe logs. You can use this information to determine the full HTTP web requests and responses that were sent when the probe failed. This file contains data only for failed probes because only failed attempts are logged. You can use this information to obtain a more complete view of why the test failed.

  - How low the availability metric dropped (x%).

  - The full path to the folder that contains the full HTTP request traces for the probe. By default, this information is located in the *\<ExchangeServer\>*\\Logging\\Monitoring\\OWA\\ClientAccessProbe folder.

  - The time and date when the alert occurred.

To troubleshoot this issue, follow these steps:

1.  Create a test user account, and then log on to the CAS by using the test user account. For example, log on by using https:// *\<servername\>*/owa.
    
    If this fails, test by using a different CA server to verify that the problem is occurring on a specific CAS, and not on the Mailbox server.

2.  Verify network connectivity between the CA and Mailbox servers. Use ping.exe to verify that each server is responding.

3.  Check for alerts on the OWA.Protocol health set that may indicate an issue that affects a specific Mailbox server. For more information, see [Troubleshooting OWA.Protocol Health Set](troubleshooting-owa-protocol-health-set.md).

4.  Start IIS Manager, and then connect to the server that’s reporting the issue to verify that the MSExchangeOwaAppPool application pool is running on the CAS.

5.  In IIS Manager, verify that the Default website is running.

6.  Locate the Mailbox Database for failed probes, and verify that the Mailbox database is active on a Mailbox server and that the Mailbox Store is healthy. To locate the failed database GUID information, open the full exception trace information. Each failure should contain an entry that resembles the following example:
    
    Starting Owa probe with Target: https://localhost/owa/, Username: *HealthMailboxdf8b87828ab0427cb91e985bbdfcec62@yourdomain.com*

7.  Copy the HealthMailbox GUID, and then run the following command in the Shell:
    
        Get-Mailbox -Monitoring -Identity <username>
    
    For example, run the following command:
    
        Get-Mailbox -Monitoring -Identity HealthMailboxdf8b87828ab0427cb91e985bbdfcec62@yourdomain.com
    
    In the returned object, you can locate the user’s database name, and you can also determine where the currently active database resides.

8.  If you have configured redirection between sites, you may see probes failing and generating a MissingKeyword error. This occurs because, by default, CA probes are run on accounts for any location, and also because the probe does not try to test a CAS on a different site when it uses redirection. To resolve this problem, make sure that the servers on each site are contained in MonitoringGroups. CA servers in a given monitoring group test only together with Mailbox servers in the same group.
    
    To determine the monitoring groups for your servers, run the following command:
    
        Get-ExchangeServer | ft MonitoringGroup
    
    To modify the monitoring group on a server, use the *MonitoringGroup* parameter together with the **Set-ExchangeServer** cmdlet. For example, use the following:
    
        Set-ExchangeServer -Identity "ServerName" -MonitoringGroup "Primary"

9.  In IIS Manager, click **Application Pools**, and then recycle the **MSExchangeOWAAppPool** application pool by running the following command from the Exchange Management Shell:
    
        %SystemRoot%\System32\inetsrv\Appcmd recycle MSExchangeOWAAppPool

10. Rerun the associated probe, as shown in step 2c in the Verifying the issue still exists section.

11. If the issue still exists, recycle the IIS service by using the IISReset utility or by running the following command:
    
        Iisreset /noforce

12. Rerun the associated probe, as shown in step 2c in the Verifying the issue still exists section.

13. If the issue still exists, restart the server.

14. After the server restarts, rerun the associated probe, as shown in step 2c in the Verifying the issue still exists section.

15. If the probe continues to fail, you may require assistance to resolve this issue. Contact a Microsoft Support professional to resolve this issue. To contact a Microsoft Support professional, visit the [Exchange Server Solutions Center](http://go.microsoft.com/fwlink/p/?linkid=180809). In the navigation pane, click **Support options and resources** and use one of the options listed under **Get technical support** to contact a Microsoft Support professional. Because your organization may have a specific procedure for directly contacting Microsoft Product Support Services, be sure to review your organization's guidelines first.

</div>

</div>

<div>

## For More Information

[What's new in Exchange 2013](https://technet.microsoft.com/en-us/library/jj150540\(v=exchg.150\))

[Exchange 2013 cmdlets](https://technet.microsoft.com/en-us/library/bb124413\(v=exchg.150\))

</div>

</div>

<span> </span>

</div>

</div>

</div>
