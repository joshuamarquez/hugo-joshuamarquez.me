+++
date = "2016-06-29T12:45:34-07:00"
draft = false
title = "How I Hacked computrabajo.com.mx"

+++

Hi community, today I will show you How I Hacked [computrabajo.com.mx](http://www.computrabajo.com.mx/),
a well known job market site in Latin America.

**computrabajo.com.mx** lets you post free jobs offers limited to the first 30 candidates,
if the number of candidates interested in your offer exceeds then you have the option to
pay for a full one to have access to the remaining candidates. Option to remove candidates
subscribed to your offer is only "available" with a full job offer but I found a
way to remove candidates from it in a free job offer allowing you to see the
candidates that were hidden due to candidate limit exceeded.

In the following image we can see that 32 candidates are subscribed to a job
offer and a message appears in the bottom telling us that 2 candidates are hidden and
can be shown paying a full job offer.

![screenshot_01](/img/screenshot_2016-06-29 12-08-15.png)

computrabajo.com.mx has a method named `DeleteCandidate(p_ims, m_folder_id)`
which is used to remove candidates subscribed in a job offer, we can see it in
Developer tools.

![screenshot_02](/img/screenshot_2016-06-29 12-11-08.png)

Now we need that `p_ims` in order to remove a candidate, we can get it from URL
by viewing a candidate profile.

![screenshot_03](/img/screenshot_2016-06-29 12-12-03.png)

Next is to execute `DeleteCandidate(<ims we got from URL>, '1')` with `ims` we got from URL.

![screenshot_04](/img/screenshot_2016-06-29 12-13-11.png)

A alert is shown asking if you want to remove candidate.

![screenshot_05](/img/screenshot_2016-06-29 12-13-40.png)

We press `OK`

![screenshot_06](/img/screenshot_2016-06-29 12-13-52.png)

Now if we list our candidates we can see that we have 31 subscriptions and a
message saying that we only have 1 candidate hidden.

![screenshot_07](/img/screenshot_2016-06-29 12-14-06.png)

Previous hidden candidate now is available.

![screenshot_08](/img/screenshot_2016-06-29 12-14-20.png)

**computrabajo.com.mx** has been noticed about this issue and we hope they will fix it soon.
