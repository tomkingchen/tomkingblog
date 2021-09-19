---
title: 'How did I pass AWS Solution Architect Professional Exam in just 3 weeks'
date: 2018-10-26T12:08:00.000-07:00
draft: false
url: /2018/10/how-did-i-pass-aws-solution-architect.html
---

This post will be a bit different from my usual technical walk through. I recently passed AWS Solution Architect Professional exam with just three weeks of preparation. In this post, I want to share my experience with the exam itself as well as how I prepared for it. Hopefully you will find it somewhat useful.

I booked the exam on 1st Oct 2018. Honestly I didn’t expect myself to pass at that time. This was the first time I tried for the Professional exam. I did my SA Associate back in April 2017. I prepared that one for over 2 months. But for this professional one, I only decided to take the exam on 1st Oct after found out the Advanced Architect course I took offered a free exam voucher. I thought to myself, this will just be a good learning experience. But somehow I managed to pass the exam in the end!

Now I won’t deny luck plays a big part, given all the answers are in multi-choice format. But there I believe are also other key approaches I took contributed to my pass. After re-track my steps before the exam, I summarized these approaches into the points below.

**Start Preparation Early**

Yes. I know I indicated I only prepared for 3 weeks, that is not completely true. While I only decided to take the exam 3 weeks before the actual exam date, I have been (casually) preparing for SA Professional since early this year. At work I have done a few AWS related projects. At my own time, I watches lots of AWS related videos, either from YouTube, LinuxAcademy or aCloud Guru. I didn't strictly follow the exam blueprint while doing all these learning. They definitely filled my brain with lots of practical AWS Knowledge. Though I didn’t expect I would be ready in such a short notice. My original plan is to take it after at least another few months.

**Subscribe to an Online Course**

Early this year I bought one year Linux Academy subscription. They offer a wide range of AWS courses, include some deep dive courses for particular AWS services like Lambda and CloudFormation (Though some of the deep dive courses are rubbish, really just “scratch of the surface” kind). I know this is not a cheap option, cost around AU$250 even after MSDN discount. But for an IT professional I think the investment is worth a while. I am not sure about you, but to me if I am paying for the course, I will make sure I take value from it. I found one effective way is to listen to the lessons while driving. Most of time, my mind doesn't catch half of the lesson. But I am good with the other half...

In addition to LinuxAcademy, I subscribed to aCloud Guru for a 7 days trial just 2 weeks before the exam. aCloud Guru has good reputation for AWS training. I passed my SA Associate mainly by doing the aCloud Guru course. I won't say their SA Pro course is the top notch. But their CloudFormation course is worth the money. It's built on real world scenarios and had plenty useful template examples. One interesting tip, I cancelled the trial before it incurs charge. But somehow I was still able to watch course videos on my iPhone app afterwards, as long as you have the course in your recent play list.

**Took Instructor Led Course**

Again this is a costly item, the training course I did is Advanced Architecting on AWS, itself cost around AU$ 2500. Mine is covered by work. It comes with a voucher for SA Professional Exam. Which I was only made aware after finished the training. The voucher expires 1 month after the training!!!

The good thing about the course is it covers services Elastic Beanstalk, EMR, which came up a lot in my exam! There are usual hands on labs plus instructor led group discussions. I especially like the group discussion session, where students form into different teams and each team work on their own version of solution to a real world design scenario. Interestingly although I am the only student doesn’t have Solution Architect as job title, I found myself became the one leading the team conversation. It appears to me most classmates don’t have a lot of real exposure to AWS.

**Hands On Experience**

Like many other IT exams, it is key to have real world hands on experience with AWS SA Pro exam. Although most questions are about high level design, with practical knowledge, you will straight away see the correct answer that makes the most sense, because you have done that in the real world! A good example is question around AWS Identity solution setup. I have setup Azure AD as AWS SSO source in a recent project. So while I saw the answers, I just picked "Setup Idp in IAM", "Setup STS", and "configure IAM role" without hesitation.

What if your work doesn't involve AWS? Well, you can create your own projects. I usually have two or three dummy projects set for myself every month. And those projects does not necessarily match the exam blueprint. I spent a lot of time writing CloudFormation Templates, which is not an essential requirement for the exam. I played with containers in ECS that has never been part of the exam blueprint. But all these hands on experience enriches my overall AWS knowledge.

**The Exam**

Above are some essential elements I think are key to my success. I would also like to share the exam experience itself, as to me it is quite unique on its own (not necessarily a pleasant one).

I have done many certification exams in my IT career for different vendors and technologies, like Cisco, Juniper, Microsoft, VMware and AWS SA Asscoiate. But the SA Pro one still came as a surprise to me. My exam is booked at HWT building in South Bank Melbourne. Upon arrival, the receptionist did not ask for any IDs, only my kiosk number. I was then led into a small room with only two computers. Without any briefing or guidance the receptionist simply showed me to the computer I supposed to work on and left the room. The "PSI exam kiosk" has two webcam built with it along with a scanner for ID scanning. Follow the screen prompts, I was then asked to scan my IDs by using the built in scanner. However, after tried number of times, the scanner wasn't able to capture my ID and I had to put the IDs up to the cams for the exam monitor to eyeball them... Yes there is an exam monitor on the other side to live monitor and chat with you during the whole exam.

After verification, I finally got to start the exam. By this stage, no instructions have been provided how you should handle the usual stuff like your phone, wallet or making notes. But anyway, I was in the exam doing it like a boss... 5 mins in, got a message from the monitor asking me to put both my hands on the keyboard. Obviously this is the rule, you can't put your hands anywhere else, but keyboard... 20 mins into the exam, guess what... the exam app crashed! I had to use hand signal to inform the monitor and he/she restarted the computer for me. Fortunately, I didn't lose my progress, although my train of thoughts are derailed number of times by this stage. 1 and half hours into the exam, another message came up on the screen, tell me to keep my eyes on the screen!!! Wonderful, I can't even take a mental break! There are number of times I got interrupted during the exam in addition to the ones mentioned above. So overall if not I passed the exam, it is a terrible experience! I don't know if it is just for this exam site or all SA Professional are carried out like this. But be prepared, in addition to the tough questions, you may have to handle some extra rubbish.

That's all the information I can provide regards the exam. Again, hopefully you can pick some useful stuff from it and good luck with the exam! And last, if not sure, just choose "C" ;)