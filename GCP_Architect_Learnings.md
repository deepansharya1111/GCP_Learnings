
<h2 align="center"># GCP Architect Learnings <br/> - Storage </h2>

<h2 align="center">1. Creating buckets</h2>

Native Google Cloud storage addresses start with gs:// <br/>
Just like web addresses start with https://

Used for:  
`Uploading & managing objects` \ 
`(Public and Private)`
   
<details>
 <summary>Storage Bucket types</summary>
 ➟ Multi-Regional<br>
  ・These buckets are designed for storing data that requires high availability and low latency access from multiple regions.
  <br>
 ➟ Regional<br>
  ・for redundancy. Regional buckets are ideal for applications that have regional data requirements, such as analytics, backups, and content distribution within a specific geographic area.
  <br>
 ➟ Nearline<br>
  ・for storing infrequently accessed data that may need to be accessed within seconds or minutes. with a slightly higher access latency. This type of bucket is suitable for data that needs to be available for quick retrieval but is not accessed frequently, such as backup archives, disaster recovery data, and long-term storage.
  <br>
➟ Coldline<br>
  ・designed for long-term archival storage of data that is rarely accessed. accessed no more than once per year, such as regulatory or compliance archives, historical records, and large-scale data archiving.
  <br>
</details>
   
---
<h2 align="center">2. Commands for storage</h2>

<pre>
- Gsutil connects to google cloud storage
- Double star lists out files even from folders 
   - gsutil ls gs://bucket-name/**
- Make Bucket
   - gsutil mb ( -l location-name) (gs://bucket-name)
- gsutil label (get gs://)[to view] 
   - (get gs://bucket > bucketlabels.json)[to export] 
   - (set bucketlabels.json gs://)[to set from file] 
   - (ch -l "key":"key" gs://bucket)[shortcut to add to existing labels]
- Check file versioning. Off by default. NOT possible on Console, only on CLI
   - gsutil versioning get gs://
   - Enable = gsutil versioning set on gs://
- View file Version with the generation-number even after its deleted
   -  gsutil ls -a gs:// 
- Make object public
   - gsutil acl ch -u AllUsers:R gs://bucket/img.jpg
     - acl = access control, -u =user, R = read 
</pre>

<h2 align="center">3. Commands for shell</h2>

<pre>
-- gcloud config get-value project
-- gcloud services list--enabled
-- gcloud services list--available | grep compute
-- gcloud compute instances create vm-name
-- screenshots  
-- gcloud compute instances list [see existing vms]
-- gcloud compute machine-types list
-- gcloud topic filters [we'll use it instead of grep so to remove paid f1 zones like north Virginia].
-- 
-- CREATING VM ⬇
-- gcloud compute machine-types list--filter="NAME: f1-micro AND ZONE~us-west" [free tier]
-- gcloud config set compute/zone us-west2-b
-- gcloud config set compute/region us-west2
-- ⮕ gcloud compute instances create-machine-type=f1-micro vm-name
-- ping-c 3 $external_ip_of_vm [name or internal ip didn't work]
-- 
-- SSH INTO VM ⬇
-- ssh $external_ip [this doesn't work]
-- ⮕ gcloud compute ssh $vm_name [this works]
-- {A public and private ssh key is now created. Public key can be viewed from Compute Engine ↠ Metadata ↠ SSH keys. That key is now registered. So any future instances they get created will automatically know to accept this for log in.}
-- 
-- CHECK SESSION IP ⬇
-- ⮕ curl api.ipify. org 
-- 
-- Where our Public KEY is saved ⬇
-- {After our first time ssh, two files: google_compute_engine (encrypted PRIVATE KEY) & google_compute_engine.pub (PUBLIC KEY) files are created into .ssh directory. Both are needed to ssh into the machine}
-- whoami ➟ owner details
-- hostname ➟ current user
-- 
-- HOW DOES VM KNOWS OUR PUBLIC KEY: ⬇
-- (run inside ssh)
-- cd .ssh/ ➟ cat authorized_keys [Contains our PUBLIC KEY]
-- curl-H "Metadata-Flavor:Google" metadata. google.internal/computeMetadata/v1/$
-- $/instance/name [our vm name]
-- $/project/attributes/ssh-keys [this contains the key that identifies us, OUR PRIVATE KEY]
-- $/project/project-id [this tells our project name]
-- $/instance/service-accounts/default/ [see info about the service account for this VM instance, like aliases, email, identity, scopes, token]
-- gcloud config list [see metadata for our project]
-- 
-- DELETE INSTANCE: ⬇
-- cloud compute instances delete myhappyvm
</pre>

<h2 align="center">4. GCP Console - Compute Engine</h2>

<pre>
When creating a Compute Instance, <br/>
⮕ In the Identity and API access section <br/>
We have our Service Account selected to give vm access to Google services. <br/>but an interesting and important thing to understand is that even when we've given access to a particular service account <br/>these access scopes restrict what you can do with it. <br/>
If you look at the "Access scopes" under Service Account,  <br/>⮕ By default you only get read access to Google cloud storage and you don't actually get access to Google compute engine <br/>which is why we were unable to terminate the instance that we were using when we shelled into it.<br/>
If you want you can "[x]Set access for each API" to send access on a service by service basis.<br/>
Or [x] Allow full access to all Cloud APIs. Which we will choose for this exploration lab.<br/>
⮕ Now if you try to delete vm after ssh-ing, you can see in the activity logs <br/>That the last two activities to delete the VM were performed by the service account being used by compute engine instance, <br/>and not our user account.
example: 339365196943-compute@developer.gserviceaccount.com deleted myvmname.

⮕ There is an option to Enable deletion protection, which is not always the best practice. <br/>
Also there is another checkbox, <br/>"[x]Delete boot disk when instance is deleted" which is ticked, <br/>because the default behavior is not to be preserving these instances, <br/>you start them up -> They do some stuff -> they go away. <br/>Going away shouldn't be a problem because the doing some stuff part should always be writing everything that matters somewhere else. <br/>
Now to store things, we can add persistent disks, but that doesn't go well with auto scaling, <br/>so you often want to use a separate service to store the data instead.<br/>

⮕ Now in the advanced settings section of compute settings:<br/>
we also have a "Security" tab and here's where we could override the project level settings about SSH keys <br/>and block them from being used and add custom ones just for this instance.<br/>
You might want to use this if you have special machines
that shouldn't be available to most people in the project <br/>or some machines where you want to allow specific access to those machines and not the others.
<br/>
Now for the disks, remember that the default behavior is to delete the boot disk whenever the instance goes away <br/>though you could either add a new one or attach an existing persistent disk that you had created. <br/>
For Key's encryption, It's generally easiest to let [x]Google manage the keys<br/>
And if possible though, you should try to avoid [x]Customer-supplied key, i.e, having to manage the keys yourself outside of Google.

⮕ For Security features, If we go to the ➟ menu ➟ to Compute ➟ look at Metadata ➟ and then switch to the SSH keys tab. <br/>
We'll see new Public SSH KEYs added every time when we ssh from the CONSOLE, as a part of connection from the console to the instance.<br/>
After running "gcloud compute ssh" the public key is put in this project metadata spot <br/>and it stored the private key on the file system that was running gcloud <br/>which happened to be cloud shell in our this case.<br/> 
But when ssh=ing from console, look at the new keys added...<br/>
They have a new "expireOn:" tag at the end, so google will automatically remove these keys in a short while. <br/>They last long enough for us to make the connection to the instance <br/>but not long enough for them to become an issue of someone stealing these credentials and using them to mount an attack.

</pre>