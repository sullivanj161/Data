1. Build it - Describe your RAID setup with supporting screenshots of your AWS EBS volumes attached to the instance and the status of your built RAID managed by mdadm on your AWS instance.
When building my RAID 6, this would require atleast 4 functioning disks. The goal of a RAID 6 array setup is to use a double-parity method of storage as opposed to just using one method of storage. It essentially allows for two disk faults to occur in one single RAID.
In AWS, I have four disks that are all attached to my one EC2 instance.
![image](https://github.com/user-attachments/assets/45f2b0d2-751d-47c9-baa5-5d493f5dcecc)

Each disk is 
20GB in Size 
Data Storage Available - According to https://www.raid-calculator.com/default.aspx I entered in that I had 4 disks, each worth 20GB for a RAID 6 double parity. Then I had calculated my results and found that my storage available would be 40 GB with Speed gain	2x read speed, no write speed gain
and a Fault tolerance	2-drive failure.
Steps taken - The steps I did to create my RAID array were as follows

1. Create volumes in EC2, for the first part of the task, I wanted to go ahead and create volumes that will be attached to my EC2 instance, as mentioned prior, I created four of them, set the capacity to 40 GB, and set their locations to be at the same location as the Instance was running at (which was at us-east-1f)
   Once these volumes were created, the next step was to was to head to the Ubuntu terminal and set up the RAID arrays for storage

2. Installing mdadm for use+
   for the sources for this, I used this DigitalOcean source to get help with creating RAID Arrays. https://www.digitalocean.com/community/tutorials/how-to-create-raid-arrays-with-mdadm-on-ubuntu

   The first step I had to do was ensure mdadm was properly installed for use on the computer
   To do this, I issued a sudo apt update first and foremost to update all ubuntu packages, then I issued a sudo apt install mdadm to log into an admin and properly install mdadm. Once this was complete, I issued a mdadm --version command
   Which came back positive and showed me a sign that there was mdadm on my server
   ![image](https://github.com/user-attachments/assets/bfb595db-0f8d-4452-9800-d3aede948088)

   3. Issue a sudo command to create a RAID 6 array
      For this step, now that we have mdadm installed, I went ahead and continued on with the steps from DigitalOcean
      The command that I issued was a sudo mdadm --create --verbose /dev/md0 --level=6 --raid-devices=4 /dev/xvdf /dev/xvdg /dev/xvdh /dev/xvdi
      This command acts as a super user to create a RAID 6 array and define its levels and directories
      For example [ChatGPT](https://chatgpt.com/) helped me break down and understand what each termm means in this command
      mdadm is the utility that is used to create and manage RAID systems
      -- create will tell the utility to create another RAID array an
      -- verbose providesa a detailed output during the creation of the RAID array
      /dev/md0 the name for the RAID array
      --level=6 signifies the level the RAID will be set up in
      --raid-devices=4
      /dev/xvdf /dev/xvdg /dev/xvdh /dev/xvdi - the disks that will be created and initialized for RAID 6

      4. Monitor the progress
         In order to check in on the progres of the RAID setup, I issued a cat /proc/mdstat

      5. Creating and mounting the new filesystem
         Next step is to create a new fileystem and mount the RAID onto it
         sudo mkfs.ext4 /dev/md0
         The ext4 will be the filesystem that will be in use for this RAID,
         Next we will want to mount it, I used the sudo mkdir -p /mnt/md0 command in order to create a directory for where the mount will go
         then the mount command I issued was sudo mount /dev/md0 /mnt/md0

         6. Ensuring the RAID is setup properly
         In order to confirm that I have setup the RAID properly, I went to ChatGPT and asked it some commands that confirm the 
         sudo mdadm --detail /dev/md0 provides a brief summary I got confirmation that it was reading in RAID6 level, the array size was 40 GB and that there were infact 4 devices in use
![image](https://github.com/user-attachments/assets/add88508-8864-42d5-b16f-696203b199b1)


Another command to use according to ChatGPT is as mentioned earlier cat/proc/mdstat which gives me the status of currently running arrays
![image](https://github.com/user-attachments/assets/60f42c7e-dc1d-4f5f-832e-8a4c11b0bd3e)


2. Detect it - How can mdadm (with or without supporting tools) detect RAID degradation? Respond in terms of the whole RAID and detection of pending failure for a singular disk.
   mdadm can detect RAID degradation which is the state of a disk failure
   I also used ChatGPT to help answer this question
   According to ChatGPT you can also use the cat /proc/mdstat to check the status of the currently operating RAID array
   ![image](https://github.com/user-attachments/assets/acdb60f6-6e95-48dd-b4a2-e78ef4c01fc4)
Circled in the image above, is the method of detecting whether or not the state of the RAID array is running fine or not
According to ChatGPT's assistance, here are what the codes will signify

UUUU - (which is what I had) means that all disks are healthy and recognizable and readable
[--UU] A disk has failed, array is in a degraded state 
[U--U] A disk has failed again, but also in the degraded state
[UU_U]: failed disk
[U-UU] failed disk as well

You can also issue a sudo mdadm -detail /dev/md0 command that will list the details of the entire RAID and give all the devices and their locations and their states 
![image](https://github.com/user-attachments/assets/c993d614-e1c2-4fee-92e8-c0e3e5ce25b5)

3. Break it - Remove a disk or two (EBS volume) used in your RAID array via AWS (not via the instance's command line). Describe the effect of this on your RAID array availability and add supporting screenshots of your AWS EBS volumes and status of your built RAID managed by mdadm.

The effect that I received on my end after removing two disks from the RAID 6 array we created.
First thing right off the bat that I noticed were the two disks that are in the volumes section, in volume state, they are now designated as "available" as opposed to "in-use".
This is because I just simply detached them from my instance, therefore, they are no longer successfully connected to my instance. With that said, I went ahead to go check my terminal for any changes
![image](https://github.com/user-attachments/assets/0529d394-7039-4d4e-963d-df173bcc4028)

The changes that I had noticed in the Ubuntu terminal  
I run a cat proc/mdstat command and notice immediately that only two drives are recognized in the raid, it is my xvdg and my xvdh drive
![image](https://github.com/user-attachments/assets/71db18a1-c8fd-4ac0-8211-29ae4e3c7f89)
It is also singifying to me the UU__ codes that interpret that their is either disabled or failed drives that exist on the RAID 

Issuing a sudo mdadm --detail /dev/md0 command I find that numbers 2 and 3 are completely removbed also and only shows 2 devices are seemingly connected. When in reality there should be 4 generally speaking
![image](https://github.com/user-attachments/assets/e7da0b4f-e91a-40de-8b2c-ac11a9c74187)

So what does this mean exactly? This means that in the even that we are running a double parity RAID 6 module, because RAID6 only allows for the failure of up to two drives, in this case since two have already been removed, this RAID is already in a degraded state hence seeing a UU__ on one of the screens. All of the data that was stored on the two drives that were previously erased are now gone. The array all in all is still usable, but however is in a degraded state. Hence when I issue the mdadm --detail /dev/md0
command, the state of the RAID is clean, degraded since two drives are now missing from this RAID. It is also important to note that RAID degradation can lead to significant performance issues in RAID arrays and drives altogether leading to sluggish read and write speeds, slow opening of programs, decreased fault tolerance of drives and potential sporatic loss of data. It is important to ensure that your RAID arrays always function properly

4. Repair it - Create new volume(s), attach them to your instance, and repair the RAID array. Add supporting screenshots of your AWS EBS volumes and status of your repaired RAID managed by mdadm.

To repair the RAID, all we would simply need to do is recreate the volumes in AWS, I just went ahead and recreated two 20GB disk drives part of the 1F area 

![image](https://github.com/user-attachments/assets/67488873-f2c4-4e78-bec1-a006d8fe40fa)

Then we want to go back into the terminal, from here, we would want to rebuild the RAID with the same commands we had initially 
At this point, I will need to mount my old filesystems, previously they were xvdi and xvdj back to md0 

For this part, I searched up which command I would use to add the created directories to the mountable filesystem. Using a post from https://unix.stackexchange.com/questions/665389/mdadm-adding-a-new-hard-disk-to-an-existing-raid1

A command that I can use to add my new disk that I had created to the already preexisting RAID array mdadm /dev/md0 --add /dev/sdc1 /dev/sdd1 is the command I can use for example to add my new drives to the array
Instead I would use 
sudo mdadm /dev/md0 --add /dev/xvdi and xvdj
This adds them
![image](https://github.com/user-attachments/assets/6e62b98a-1443-42c4-91c4-27292b10393f)
After this process, I want to ensure that the rebuilding process is working, so I use a cat /proc/mdstat which shows progress
![image](https://github.com/user-attachments/assets/bd187585-d339-4d68-8c54-a88732709eb8)
Now that the rebuilding process has complete, I issue one more cat /proc/mdstat command that recognizes the UUUU drive code that 4 drives are properly installed, in the MD0 tab, I can see plainly that 4 drives are clearly active.
![image](https://github.com/user-attachments/assets/42e96bc2-3a4b-44a0-b609-e426eea6c8e8)
Issuing a sudo mdadm --detail /dev/md0 command
I can now see that the state is no longer degraded, since I have more than 2 active drives on this RAID array
The state is clean, there are 4 active sync state drives in the bottom and recognizes the total devices, the UUID and the RAID level of the array.
![image](https://github.com/user-attachments/assets/da3f9522-21ba-4a82-ad86-3ed5658f1d9c)








         
