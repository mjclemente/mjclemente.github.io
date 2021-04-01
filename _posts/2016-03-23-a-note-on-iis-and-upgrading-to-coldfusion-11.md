---
published: true
title: A Note on IIS and Upgrading to ColdFusion 11
layout: post
tags: [coldfusion,cfml,iis]
---
We recently upgraded to ColdFusion 11 (I know, less than ideal timing). The process was relatively smooth because we ran ColdFusion 10 and 11 side-by-side before completing the upgrade. I don't think we took this approach when moving from ColdFusion 9 to 10, but it's actually fairly easy to do. We only ran into one complication - an IIS configuration issue - that had a simple solution, and also helped deepen my understanding of how the application server interacts with IIS.<!--more-->

Yieng Ly's blog post documenting the process of [updating from ColdFusion 10 to ColdFusion 11](https://yiengly.wordpress.com/2015/04/18/upgrading-to-coldfusion-11-from-10/) was very helpful. Obviously, Pete Freitag's [ColdFusion 11 Lockdown Guide](https://www.adobe.com/content/dam/Adobe/en/products/coldfusion/pdfs/cf11/cf11-lockdown-guide.pdf) is also essential. Following Yieng's steps, the CF11 installation process went smoothly. 

Our problem arose while setting up a temporary IIS website for ColdFusion 11. We're using IIS 8.5, for the record.  ColdFusion 11 was installed, and we configured the connectors for the specific temporary website, but when we tried to access it, we got an error:

![ColdFusion 11 Installation 500 Server Error](/public/assets/images/coldfusion-11-installation-500-server-error.png)

Some poking and prodding under the hood revealed that, even though we had run ColdFusion 11's Configuration Wizard for the single site, the site's IIS Handler Mappings for CFML file types, were still pointing to ColdFusion 10:

![ColdFusion 10 IIS Handler Mappings](/public/assets/images/coldfusion-10-iis-handler-mappings.png)

Correcting this issue alone, however, did not solve the problem. After restarting ColdFusion 11 and IIS, we continued to get 500 errors - because, drum roll please, the ISAPI Filter for *tomcat*, for the website, was still pointing to ColdFusion 10. This setting, which is inherited from the IIS server setting, doesn't look like it can be changed within the IIS GUI at the individual website level. So, to point the individual website to ColdFusion 11's *tomcat* ISAPI Filter, while leaving the rest of the sites using ColdFusion 10's, we needed to edit the `web.config` file for the site: 

```xml
<isapiFilters>
	<remove name="tomcat" />
	<filter 
		name="tomcat" 
		enabled="true"
		path="C:\ColdFusion11\config\wsconfig\1\isapi_redirect.dll" />
</isapiFilters>
```


This overrides the IIS server's tomcat ISAPI Filter with a local entry for the website. A quick restart of IIS again, and the issue was solved, no more 500 errors. We were able to move forward with configuring ColdFusion 11 through the temporary site:

![ColdFusion 11 Configuration and Settings Migration Wizard](/public/assets/images/coldfusion-11-configuration-and-settings-migration-wizard.png)

So, in summary, to run a different version of ColdFusion for a website, we needed to update its 1) IIS Handler Mappings and 2) ISAPI Filters. I imagine the process would be similar for any other upgrade of ColdFusion, or even for setting up Lucee for individual websites. I'll probably end up with more posts on that, when we get around to diving deeper into Lucee as an alternative to our current setup.



