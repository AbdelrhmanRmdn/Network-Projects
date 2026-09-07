# Network-Projects
Hey there! 👋 I'm Abdelrahman, an IT student with my eyes set on the Blue Team.

I figured out pretty quickly that to defend a network, you first need to understand how it actually ticks—and get really comfortable living in Linux. So, I set a challenge for myself: 30 days, 30 networking projects.

In this repo, I'm sharing the journey raw and real—every project, what I actually learned, and the roadblocks and bugs that drove me crazy along the way.

==============
    Day 1
==============

The setup : 
  2 Pcs
  1 switch
  
Experience:
I connected 2 PCs through a switch i setup the IPs manually for PC0 i used IP ( 192.168.1.10) and Subnet ( 255.255.255.0 ) and for PC1 IP ( 192.168.1.11) and Subnet ( 255.255.255.0) and i ping PC1 by using the command prompt of PC0 and i worked good then i tried to change the subnet to see what will happened i changed it into (255.255.255.128) I found out that it worked so good like i didn't do any changes bc in (255.0 it has block include from 0 to 255) and ( 255.128 has 2 blocks include from 0 to 127 and 128 to 255) so i saw that there is a common range and i looked at both of the IPs its between the two ranges ( 0 -> 255 & 0 -> 127) so it pings normally but when i changed the IP of PC2 into 192.168.1.130 it didn't work bc it will use the second block of 255.128 subnet and i will be out of range to the IP of the first PC

What i learnt:
1- What is the subnet and what does it actually do
2- what happen if you change the subnet of one pc and keep the IPs the same
3- What happen if you change the IPs of one of the PCs
4- The block/range rule 

Challenges:
i didn't understand the block/range rule at first it was so confusing but then i divide it into so many parts then i understood it 

conclusion: 
it was a funny Experience and im so excited for the other days 


