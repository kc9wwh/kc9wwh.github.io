---
title: "The First Issue with Starting a Blog"
description: "Is that you don't just have to make time to do the projects, but also to write about them!"
date: 2025-01-24T16:10:38-06:00
image: cover.jpg 
categories: [Tech]
tags: [Windows, Entra ID, Intune]
hidden: false
comments: true
draft: false
---
If you've ever made an effort to do a blog, you'll understand. The real fun is learning about new things, but you also need to prioritize writing about the things you do as quickly as possible so that you can provide more detail around the things you do and be able to better talk about the challenges you faced and how you got past them. Bottom line, the longer you wait, the more it'll show. 

So, starting today, I've made a commitment to do better. 

Until very recently I was drowning in Apple devices. But after taking a short professional break, I was left with only my Apple iPhone 15 Pro Max and my ASUS ROG Zephyrus gaming laptop with at the time Windows 11 Home. Since then I've replaced the cooling fans and reapplied thermal paste, as well as upgraded to Windows 11 Pro. I do have to admit that there were a lot of frustrations early on with moving all in with Windows as my daily driver. To be clear, there are still some, but overall in the last few months, Windows has started to grown on my. 

Now the largest hurdle was understanding that the Windows world loves to access their email via a web browser or Outlook. I love Apple Mail and the variety of third-party email clients available on macOS. But for whatever reason, the number on the Windows side is much much lower and so is the ascetic design qualities I've come to appreciate in the Apple ecosystem. I've tried anything that stood out and looked great, but in the end, there was always something off, something I didn't like about it. In the end, I choose to settle with a mix of Outlook and more and more I finding myself accessing my iCloud Mail account via the web browser. This is most likely a personal preference, but for me, it was the hardest part to overcome and the lack of Apple to Microsoft integrations don't help.

As I started using my computer more and more, I have gotten really comfortable with Windows again, even though I really haven't done much technically with a Windows computer since Windows XP. That said, I am absolutely loving two big things for me as I dive more and more back into the technical side on Windows. First, I love that Microsoft has brought Linux to the platform via WSL (Windows Subsystem Linux). This allows me to access any command line utilities I was already accustomed to from working in macOS and Linux systems. On top of that, Microsoft has created a kind of redirect in Windows Command prompt and Powershell where you can use ls, rm, etc and it just "redirects" it to the appropriate Windows module and has (limited) flag support for them as well. But this has certainly made my switch much easier. 

Secondly, I absolutely love and was thrilled to find that [Oh-My-Posh](https://oh-my-posh.dev) had support for Windows! I'm using it right now in Powershell with VIM writing this blog post in markdown and I absolutely love it! I'll also say that Powershell is very intuitive and the modules are very well documented for those occasions where you want to write some Windows scripts. 

Today, I had the opportunity to work on a side project involving Entra ID and Microsoft Endpoint Manager (a.k.a. Intune). Now I have some previous Windows experience from when I was growing up and early on in my professional career. However, I remembered everything being scripted. So, while I was waiting for my Azure access to go through, I started planning to deploy a Powershell script to newly enrolled organizational devices. This script would install Google Chrome, Slack, and Microsoft Office 365 along with configuring a SharePoint sync through OneDrive and mapping a network drive. 

But, when I received access to Azure and logged in, I started to realize that a lot of the features and comforts I was familiar with on the Apple MDM side, has made their way over to Windows as well. Deploying apps from the Microsoft App Store is straight forward and quick to setup. You can also use a tool called the [Microsoft Win32 Content Prep Tool](https://github.com/microsoft/Microsoft-Win32-Content-Prep-Tool) to create a deploy-able application from an .exe or .msi installer. Which, I did use to create a Google Chrome Enterprise package which I uploaded to Microsoft Endpoint Manager and was able to successfully deploy. 

Configuration Profile or Configuration Policies (as they are referred to in Microsoft Endpoint Manager) provide the same type of functionality restrictions and configuration capabilities you'd come to expect on the Apple MDM side as well. Even better, Microsoft Endpoint Manager has the most common [Configuration Service Providers or CSPs](https://learn.microsoft.com/en-us/windows/client-management/mdm/policy-configuration-service-provider) already loaded into the platform, so you just need to essentially search for what you are looking to configure, such as setting a home page in Google Chrome or configuring OneDrive to sync with SharePoint. Once you found the key you were looking for, just add it to the Policy, enable the key and configure and needed parameters associated with it. Next, define your scope for how, when and to whom you are going to deploy the Configuration Policy and you're done. 

> Quick Note: From my (all be it limited) experience with Windows MDM, some of these policies will not apply until a restart or the user logs out of the device. 

Once I had the various applications ready to go in the Company Portal, along with a few Configuration Policies configured, I decided to take a look at branding the Company Portal to really make it feel as part of the organization. But, I was unfortunately disappointed at the available options for really applying a custom brand to the application. I was able to specify a custom color for the application, as well as logo and a few other minor things, but the changes on the Company Portal side were extremely minimal, mostly just changing the menu bar of the window. The rest of the applications look stayed the same and really clashed, so I ended up deleting the branding config and left it with the default. 

Anyway, on Monday its back to testing and improving the build process for when we have to reload a system or setup a new device for someone. 

Till Next Time!

Josh 🖖

> Photo by [Jerry Kavan](https://unsplash.com/@jerrykavan?utm_content=creditCopyText&utm_medium=referral&utm_source=unsplash) on [Unsplash](https://unsplash.com)
 
