
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
-- {FYI: after first time ssh, two files: google_compute_engine (encrypted private key) & google_compute_engine.pub (public key) files are created into .ssh directory. Both are needed to ssh into the machine}
-- whoami ➟ owner details
-- hostname ➟ current user
-- 
-- HOW DOES VM KNOWS OUR PUBLIC KEY: ⬇
-- (run inside ssh)
-- curl-H "Metadata-Flavor:Google" metadata. google.internal/computeMetadata/v1/$
-- $/instance/name [our vm name]
-- $/project/attributes/ssh-keys [this contains the key that identifies us]
-- $/project/project-id [this tells our project name]
-- $/instance/service-accounts/default/ [see info about the service account for this VM instance]
-- gcloud config list [see metadata for our project]
-- 
-- CHECK SESSION IP ⬇
-- ⮕ curl api.ipify. org 
-- 
</pre>
