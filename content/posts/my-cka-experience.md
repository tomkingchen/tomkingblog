---
title: "My Cka Experience"
date: 2024-09-27T21:02:22+10:00
draft: false
---

Recently I was able to tick CKA off my TODO list. It’s a interesting and unique exam in compare to other certs I have done in my career. Below are some points I want to share to help out folks who plan to take the exam.

## Was it hard?

The exam is not super difficult in my opinion. My daily job gives me plenty of exposure to different Kubernetes issues. So I was relatively comfortable with the technology itself. On the other hand, I don’t think it is an easy exam based on following factors.

- Unlike a lot of IT certification exams, the questions you get in CKA are not in multi-choice format. So no WhizLab cheats! Instead you are given individual tasks that has to be completed by typing commands in a Linux terminal. This does not only mean you will need to remember and understand a lot of the Kubernetes related commands, but also this format creates plenty of rooms for mistakes to happen! For example, a single typo in the resource name will ruin all your hard work for a task!
- CKA booking doesn’t provide designated test centre option. So you have to find suitable location yourself. There are a few annoying but must meeting criteria for exam location. One key requirement is that you cannot have clutters in the room. That means if you have a picture of your family on the wall, you cannot use that room!
- Since they don’t provide test locations, you will need to use your own device for the test. Just before the exam you are required to download and install a special browser to access the remote exam session. Depends on your network settings, you might have issues access the Internet through the browser. E.g. corporate proxy settings. Therefore it is highly recommended you use your own personal device instead of your company laptop.

## Learning resources

I started my Kubernetes learning with a Udemy course from [kodekloud.com](http://kodekloud.com) taught by Mumshad Mannambeth. The course covers all the exam topics with right amount of depth. The free practice labs and mock exams from Kodekloud portal are very well made and does help me to be more familiar with the exam style.

In addition to the online course, I spent a lot time on [killercoda.com](https://killercoda.com/cka) to go through the 120+ CKA scenario there. These scenario does not necessarily matches actual exam questions, but they are good tests to assess my knowledge level on a particular topic and further enhance my familiarity of those kubectl commands.

Another useful site is [killer.sh](http://killer.sh). The site provides simulators that are very close to real test environment. The remote desktop session in the simulator provides almost like for like exam experience. But you do have to purchase the simulator sessions. I don’t feel the questions asked in the simulator are more relevant than those scenario from [killercoda.com](http://killercoda.com). But to me it is worth the cost just so I can be really familiar with the exam environment. 

Apart from the above resources, I did spend sometime to setup my own Kubernetes cluster at home. Setting up things from scratch can always help to better understand a technology. You might bump into unexpected issues and provides opportunties to dive deeper into the area usually not covered by online courses.

In addition to all above, my daily role provides plenty of challenges related to Kubernetes. From onboarding workloads to troubleshooting pod crash issues, these knowledge all help to prepare me for the exam.

## Exam day

To ensure there won’t be any issues with the exam location, I booked a “focus room” at work for 2 hours. I arrived half hour earlier before the exam to set thing up. Lucky I did that, it took me a while to finally make the webcam work with my laptop. After that, I had to chuck my water bottle out of the room since it is not a transparent plastic bottle. 

The exam itself took 120 minutes. I found myself had around 40 minutes left after completed all the questions. Although there are around a few questions I flagged as unsure. One issues I found during my review is that I forgotten to switch cluster contexts in a few occasions! With CKA, there are normally more than one cluster context used. You really need to be careful and double check the cluster context before jump on the task. I was able to redo some of those tasks on the correct context.

So overall, I didn’t have too much issue with timing. I believe that’s largely thanks for all the practice scenario I did before the exam. I was very unsure if I will pass. For CKA the result is not given straight after the exam, but within 24 hours. I didn’t receive the result until next day morning. 

## Tips

Here are some tips I think helped me during my exam.

- Practice the commonly used kubectl imperative commands like `kubectl run` and `kubectl expose`.
- Common alias like `k` for `kubectl` is setup in the terminal already.
- Familiar yourself with useful Linux commands/tools like `grep` and `awk`.
- Copy and paste everything from the questions to the terminal during the exam. No one wants to fail the exam just because some silly typos!
- Remember to switch cluster context before each question!
- Flag the questions you are not sure about and move on. You can always come back to them later.
- Double check the status of resources you created. Just so you can be sure things are created as they should be.
- Double check the namespace and make sure you are in the right one!
