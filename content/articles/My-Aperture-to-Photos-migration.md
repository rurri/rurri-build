---
title: "My Aperture to Photos migration"
posted: 2015-05-16T14:27
tags: [aperture, photos, osx]
track: Photo
---
I took the plunge and decided to migrate from Aperture to Photos. I really like taking photos, and like to pretend that I’m a better photographer than I am. To this end, for many years I have used Aperture for Mac OS X to grab photos of my camera then tag, rate, organize, and do some basic brightness/contrast adjustments to the ones that really needed it.

I have liked Aperture, and it has served well, but it has had a two major competent missing for me:

### Built in sharing with others / syncing to devices ###
This can be mitigated by exporting albums or other sets of photos to Facebook or other location, but it is not as smooth as it should be. Especially if you later go back and enhance and crop photos later, trying to sync this change back into Facebook or other photo sharing site is a non-starter, and will require a manual workflow.

In the past I had used [Phanfare](http://www.phanfare.com). It is a pretty good way to store originals and share them. I was on their lifetime version back before they ended it. This is still a decent option at $99 a year, but you still have to export from Aperture to Phanfare, so the experience wasn’t the full integrated solution I was going for.

Lastly getting these photos on my mobile devices to share with others was just one more manual step in the workflow I had to do. Pick the photos I might want to share with people and make sure it was either in Facebook or manually copy them to my phone / iPad. Not ideal at all.

### Integrated cloud backup ###
A major part of bringing my photos into aperture for me has been getting them into my backup workflow. In aperture a backup workflow consists of manually managing “Vaults”. I’ll be the first to admin that I have been slow to learn the details and nuances of how these work. I learned enough to back up my photos onto external hard drives which seems to be their primary purpose, but I could not do any advanced work with them.

One thing I have always wanted was to backup photos in the cloud. I don’t like managing all these hard drives. I don’t trust myself not to lose them. I would feel much better with at least one copy of each photo sitting on a cloud service instead.

For me, in Aperture this meant storing photos as referenced files, and then syncing the referenced images in a location that got synced to Google Drive. This could just as easily been Dropbox or other, but I chose Google Drive. Then the Aperture Library itself could then be sent to Google Drive now and then as a backup. I found trying to sync the entire Aperture library with images being inside the library was too much for Google Drive to handle, and the entire library would need to be uploaded in the event of a small change or just a photo or two being imported.

In short, many ways exist to get these files backed up, and they worked, **but** hey all required manual workflow on my part. Many times I would fall behind and then spend my free time getting all these backups caught up. So not a lot of fun trying to manage all these processes.

## Why I switched to Photos ##
Simply put, Photos solves these problems magnificently. 

With iCloud turned on in OSX, photos are automatically synced to the cloud, and built in sharing allow you to just select an album or photos, and share it with people on your contact lists. 

Files will be removed locally when storage gets tight on your Mac and kept only in the cloud (This can be disabled). Sharing is quick and doesn’t require uploading to any social media site and allows for quick and private sharing of photos without re-uploading them anywhere—remember they are already stored in the cloud by default). This means sharing them takes no time at all, since the files don’t have to be uploaded anywhere!

Photos taken on my phone are also automatically a part of my library both in the cloud and on my Mac. If I don’t have my camera with me for a while or want to travel light, I can just take my phone and no imports are required at all.

Even better is if you have a wifi-enabled camera and it is set to send taken photos to your phone, you never have to offload photos from your camera again. As you take the photos, they are sent to the cloud, and subsequently synced and downloaded to your Mac. I don’t have this yet. Perhaps a future camera upgrade will let me do this!

The bottom line is Photos completely simplifies my workflow in all cases. Photos are sent to the cloud with no input, photos can be shared quickly, photos end up on all of my devices without any input from me. This is exactly what I want—no workflows. I think the fact that we talk about workflows when we talk about managing our photo libraries means there is a fundamental problem.

For me, Photos eliminates all my workflows.

## Losses from Aperture ##
There are a number of features that get lost when switching from Aperture to Photos. Photos is not nearly as advanced as Aperture in terms of professional features. Here are a few items it is missing. Granted these are things I noticed in my workflows and is in no way exhaustive.

### Photo editing ###
The tools for editing in Photos are not as near advanced as they were in Aperture. If I was doing more than Brightness/Contrast adjustments I would have been in a world of trouble.

Photos currently limits you to adjusting brightness/contrast, applying filters, and other basic photo adjustments. This is not great, but has not been a problem for me yet. If I am going to use the image in some sort of printed matter I usually end up pulling into Photoshop first anyways and doing more detailed touch up there, so the quick controls have not been a problem (yet), for doing quick adjustments prior to sharing on Facebook or other site.

This is probably the biggest loss and downgrade to Photos in my opinion.

### Backups ###
The cloud is the backup. I recognize this can be a turnoff for some. It is of course its biggest feature for me.

The problem is, other than manually backing up the library and referenced files (completely an option) there are no built-in backup 

### Star Ratings ###
I was a big user of star ratings. I like slowly whittling down my photos to 2-star, then taking those and whittling them down to 3-star, etc after each import. In the end each photo would have zero to a few 5 stars that were at the top of my list. This feature is completely removed in Photos.

In Photos you are left with a heart icon and you can either favorite it or not. At first this stings, since I spent so much time managing my star ratings, but perhaps the time I was spending trying to manage these star ratings was not worth the payback in my quick searches. Time will tell.

During the conversion from Aperture to Photos. Photos tagged each photo with its star rating so this information on old photos is not lost. A 2-star rated photo is now tagged with “2 Star” so this means all the work is not lost. Indeed, if I wanted to try and continue using the stars just as tags I guess I could, but I will try the more simplified version of Favoriting and see where it gets me. My guess is it will save a ton of time after a camera import, and finding my top photos still won’t take a lot of time.

## Snags in the conversion ##
While I love the Photos experience with my workflows over Aperture, what is terribly disappointing has been the conversion process from Aperture.

Here are some lessons I have learned in the process.

### Backup Backup Backup ###
I know this goes without saying, but this whole process is so bad, I had to do it several times. **Do not start this process without backing up, and starting with a copy you can afford to lose**

### Referenced Files not handled well ###
With my previous workflows, having referenced files was the way to go with backups. Photos does not handle referenced files well. Its UI is *terrible* at locating referenced files that have moved.

The easiest way to address this is to consolidate all your originals *before* the conversion. This will save a lot of trouble later.

This was a sore spot for me to begin with since my previous workflow required photos to be referenced. To be sure I was not overly committed I consolidated the originals on a copy of my original aperture library.

**Consolidating Originals prior to conversion is highly recommended**

### Upload takes long time and sometimes stalls ###
My first shot at syncing to the cloud did not go well. In fact nothing uploaded. The preferences screen showed: “Uploading….” without any movement, when it should have been showing how many photos were left to upload. I had to delete my converted Photos library and start over again.

The uploading takes a *long* time. For me it took over 3 days. This is a problem in that with the initial sync not so solid, and the possibility of having to start over a real possibility. This means there should not be any enhancing photos or adding new photos, since a restore from the backup would wipe out all the new photos and changes.

**Initial upload takes a very long time to sync to the cloud be prepared to not work on your photo library for several days**

### Smart albums aren’t converted perfectly ###
This is not a major problem, but something that should be checked before accepting that everything converted correctly. In my case I had a smart album called “2 Star and higher”, which in Aperture was all photos from a specific album that were 2 star and higher. Photos converted this to be photos tagged as “2 Star, 3 Star, 4 Star, or 5 Star” which correctly pulled only the 2 star and higher photos, but it did not filter on a specific album.

I didn’t have a great deal of these so it was not a very large problem.

**Check smart albums to verify they converted correctly**

##  The new experience ##
So far I am really loving my new Photos experience over Aperture. Maybe I was just never that much of a professional photographer, but I absolutely love that I don’t have a workflow.

I add photos either by importing from my memory card or they get added automatically when taken from my phone.

I share photos if I want to with a single right click, and no additional upload needs to happen. They are just shared.

My photos are backed up by both the iCloud and my TimeMachine.

These are all things I don’t have to think about now, meaning now I am without a workflow.