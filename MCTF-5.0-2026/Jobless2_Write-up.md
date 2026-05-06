# Jobless 2

**Category:** OSINT
**Flag:** `mctf{LARGO-1995_DANIELLE-GRAEF_83_THE-ZOHAN_22/03/2006_ZZ9}`

---

**Challenge description:**

![](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/image1.png)

> Andrei is a hardcore fan of the author of the book that he told Stay about. He stalked him day and night, and could extract some interesting information about him. Can you do that as well?
>
> Important Note: This challenge is obviously a sequel of Jobless 1, so you need to solve the first part to be able to solve this one.
>
> Flag Format: Part 1: The city where he lives and the year he moved to it. Part 2: The name of a person that works in his business. Part 3: His age. Part 4: His favorite movie. Part 5: The date of when was the attached picture taken by the author (DD/MM/YY). Part 6: The first 3 characters of his license plate.
>
> The flag is all in Uppercase, put "\_" between parts and "-" inside parts if they contain more than 1 word.
>
> Example: mctf{CHICAGO-2007_STAY-HERE_60_TOY-STORY_30/04/2026_ABC}

The challenge also ships with a downloadable attachment called `bridge.png`.

It is a photo of a wooden boardwalk cutting through beach grass and leading straight toward the water on a clear sunny day .... first glance it looks like any random beach in Florida which is not very helpful.

![](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/image3.png)

---

**Tools used:** Google Maps, theresumestore.com, Flickr, LinkedIn, Twitter, Instagram, Facebook, Blogger, WordPress, Truthfinder, Trustpilot, Google Search.

---

This challenge is a direct continuation of Jobless 1. The final step of Jobless 1 is finding a Google Maps listing for a place called **Get A Job Manual**, dropped at coordinates somewhere in the middle of the Pacific Ocean.

Obviously not a real business location, just a hiding spot. Inside the listing you find the Jobless 1 flag and more importantly a link to a website: **theresumestore.com**.

that website is the bridge between the two challenges and it is where everything in Jobless 2 starts.

![](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/image4.png)

The [Maps listing](https://www.google.com/maps/place/Get+A+Job+Manual/@48.6170322,-153.3785226,15016180m/data=!3m1!1e3!4m18!1m9!3m8!1s0x88c2f122c487f231:0x70a74d6ae55c63b2!2sGet+A+Job+Manual!8m2!3d46.423669!4d-129.9427086!9m1!1b1!16s%2Fg%2F11h4w9sk47!3m7!1s0x88c2f122c487f231:0x70a74d6ae55c63b2!8m2!3d46.423669!4d-129.9427086!9m1!1b1!16s%2Fg%2F11h4w9sk47?entry=ttu&g_ep=EgoyMDI2MDQyOC4wIKXMDSoASAFQAw%3D%3D) itself shows a few details worth noting.

The place is categorized as a "Resume service" with a 5.0 rating from four reviews.

The website link is theresumestore.com and the phone number listed is +1 727-219-0177.

Two images are attached to the listing, a woman in professional attire posing for a headshot-style photo. There is nothing else of value in the listing itself. The website is what matters.

---

**Opening the website**

![](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/image%209.png)

The first time you load [theresumestore.com](https://theresumestore.com/) you are greeted with what can only be described as an aggressive amount of text.
The homepage is long and dense, pushing the idea that Arnold is the best resume writer in Florida. He claims that 85% of clients he writes resumes for go on to get job interviews. There are sections with statistics, bullet-pointed benefits of hiring a professional resume writer and multiple video embeds that are supposed to be testimonials.

Reading through the content it is pretty clear that most of the body text is AI-generated.

What we actually paid attention to was the footer and the about section.

The footer lists the business location as **Largo, Florida, USA** and includes a line about being in business since 1995. The about section elaborates slightly.

Arnold positions himself as a career consultant who has been working in the field for decades, originally out of New Jersey, and moved his operation to Florida. The site also lists four social media accounts: LinkedIn, Twitter, Instagram and Facebook. All four are branded under "The Resume Store" rather than his personal name.

![](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/imaeg%208.png)

At this point we had Part 1. The city is Largo, the year is 1995. Both are stated directly on the site.

**Part 1 answer: `LARGO-1995`**

---

**Going through the social media accounts**

We opened all four accounts at the same time and started going through them in parallel.

The **[Twitter](https://twitter.com/StoreResume)** account (@StoreResume) is fairly inactive. Most of the posts are links back to the website or short generic career tips along the lines of "make sure your resume has a strong summary section" and "keywords matter more than you think."

There are no personal posts, no photos of Arnold, no mentions of anyone else. It reads like a feed that was set up to look active but nobody really uses. We scrolled to the bottom. Nothing useful.

![](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/twitter.png)

The **[Instagram](https://www.instagram.com/theresumestore/)** account (@theresumestore) is more active but in the same vein.

The **[Facebook](https://web.facebook.com/TRSResumes/)** account (TRSResumes) was similar. Business-focused content, links to the website, some longer posts about resume trends.
We scrolled through the entire timeline. A few posts had photos attached but they were all stock images or the some AI-generated posts.

No bridge photo, no personal images, no mention of any employees or collaborators.

![](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/fb.png)

The **[LinkedIn](https://www.linkedin.com/in/theresumestore/)** account had a bit more substance.
The profile listed the business as a small company with around 2 employees operating in the career coaching and resume writing space.
Arnold's own LinkedIn shows his experience going back to the 1990s. There is a recommendations section with a few clients saying positive things and a skills section listing resume writing, career coaching and LinkedIn optimization.
Nothing here solved any of the remaining parts on its own but we noted the employee count and kept it in mind for Part 2.
![](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/linkedin.png)

---

**Part 2 — finding who works with him**

Part 2 asks for the name of a person who works in his business. The website has no team or staff page. None of the social media accounts mention an employee by name. We were a bit stuck.

We tried a different approach. We had found a copy of Arnold's own CV floating around online. The filename was something like `ARNIESHERRPROFESSIONALRESUMEWRITERRESUME`, a resume writer who put his own resume online which is kind of funny.

![](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/resume.png)

Reading through it his work history goes back decades. One entry that stood out was time spent at **Cumberland Farms**, a convenience store and gas station chain that operates in the northeastern United States.

We went and looked up [Cumberland Farms on encyclopedia.com](https://www.encyclopedia.com/social-sciences-and-law/economics-business-and-labor/businesses-and-occupations/cumberland-farms-inc) which has a business profile page for the company. The page covers the company's founding history, expansion, ownership structure and operations.

We read through it thinking maybe the connection between Arnold and Cumberland Farms would lead somewhere, maybe a named colleague or manager. It had general corporate history but nothing that linked to Arnold specifically or named anyone relevant to the flag. That path was a dead end.

We went back to LinkedIn and searched for "The Resume Store" then filtered by people who had it listed as their employer. That gave us the full profile we were looking for: **Danielle Graef**.

![](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/image7.png)

Her headline reads: "Financial Customer Service Representative | Banking Operations | Account Management | Transaction Accuracy & Compliance | Cash Handling & Reconciliation."

She is based in Clearwater, Florida which is directly adjacent to Largo and consistent with working for a business there.

Her listed employer is The Resume Store and her education shows St. Petersburg College. The arrow in the screenshot above points directly at The Resume Store logo under her name.

LinkedIn: [Danielle Graef](https://www.linkedin.com/in/danielle-graef-financial-csr/)

Later on we also cross-checked by going directly to the [Resume Store people section on LinkedIn](https://www.linkedin.com/company/the-resume-store/people/)

The people section listed her under the company right alongside Arnold, which gave us a second confirmation she was the right answer.
![](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/Untitledlinkedin.png)
**Part 2 answer: `DANIELLE-GRAEF`**

---

**Part 3 — his age**

We turned to public people-search databases. Florida has relatively open voter registration records and several third-party directories aggregate that data.

Searching for Arnold Sherr in Largo, Florida on [Truthfinder](https://www.truthfinder.com/report-review/?utm_source=WXAB&traffic%5Bsource%5D=WXAB&utm_medium=affiliate&traffic%5Bmedium%5D=affiliate&utm_campaign=FL&traffic%5Bcampaign%5D=record%3AFL&utm_term=more_about&traffic%5Bterm%5D=more_about&utm_content=driving-records&traffic%5Bcontent%5D=driving-records&s1=FL&s2=record&s3=more_about&s4=driving-records&s5=&traffic%5Bfunnel%5D=bg&ck_rsid=4006457186&firstName=Arnold&lastName=Sherr&city=Largo&state=FL&age=83&tcg_id=1151362b-7369-4253-b275-998142e772d5&transaction_id=f1a9089a-5a60-465e-aeec-26cfc2799c96&index=1&bestResult=false&intent=curiosity) returned a detailed entry:

![](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/image6.png)

The listing shows Arnold Sherr, age **83**, born February 2, 1943, currently listed at 250 Rosery Rd Nw #215, Largo, 33770-1208 Florida. He is affiliated with the Florida Democratic Party. The page also notes a possible relative named Brian David Sherr, age 58, born August 1967. His phone number is partially visible as (727) 796-0\*\*\*.

This matches the Largo address we already had from the website. Age 83, born 1943.

**Part 3 answer: `83`**

---

**Part 4 — his favorite movie**

This one genuinely took a long time and involved a lot of dead ends.

The obvious first move was to check the social media accounts again with fresh eyes, specifically looking for any personal content, any post where Arnold talks about something he likes, watches or does outside of work.

We went back through Facebook post by post. The feed goes back several years and there are hundreds of posts. All of them are resume tips, career graphics and client success stories.

.... Not a single personal post.

We tried the [WordPress blog](https://theresumestore.wordpress.com/). The blog has a handful of posts, most of them career advice articles about how to structure a resume, what to include in a cover letter and how to handle salary negotiation. The writing style is similar to the main website, competent but generic. There are no personal posts, no off-topic entries, nothing about movies or hobbies. We read through all of the posts. Nothing.

At this point we had been looking for the movie for a while and were going in circles. We tried a few Google searches combining his name with words like "favorite" or "movie" or "review" but nothing useful came back.

Eventually we changed approach and started searching for older web presence. People who have been online since the early 2000s often have accounts they forgot about, forums, older blog platforms, Geocities-era stuff. We searched specifically for "Arnold Sherr" on Blogger and found a profile:

[https://www.blogger.com/profile/10356161715111330278](https://www.blogger.com/profile/10356161715111330278)

![](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/image5.png)

The profile has a full "About me" section. The details are interesting on their own. He lists his occupation as Writer, his location as Clearwater FL and notes he has been on Blogger since June 2008 with 677 profile views. He describes himself as a news junkie who writes for the Tampa Bay Informer newspaper and mentions political interests. His favorite music section mentions Sinatra and Sirius satellite radio.

The key entry is under **Favorite movies**:

> "Lou Dobbs, CNN, CNBC - oops; Sorry, you asked about favorite movie(s). At present there is but one; 'The Zohan' was great!"

He starts by accidentally listing news anchors then corrects himself and names The Zohan.

**Part 4 answer: `THE-ZOHAN`**

---

**Part 5 — date the photo was taken**

We needed to find the original source of `bridge.png` to get the date it was taken. The image is a wooden boardwalk leading toward a beach, the kind of shot someone takes on a walk, a personal photo rather than a stock image.
![](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/bridge.png)

The next step was reverse image searching. We ran the image through a few tools. The results were not great, similar beach boardwalk photos came up but nothing that matched exactly. We did not find a clean direct match through that method.

We went back to the social media accounts and looked specifically for photos rather than graphics or reposts.

On a hunch we searched for Arnold Sherr on Flickr. Flickr was a major photo sharing platform in the mid-2000s and many people from that era still have old accounts sitting there with photos from that time. His account is at [flickr.com/photos/sherrent](https://www.flickr.com/photos/sherrent/).

The account has multiple uploads, mostly personal photos from Florida, beaches, local scenery, that kind of thing. We went through the gallery and found the boardwalk photo. It matched the `bridge.png` attachment from the challenge exactly, same composition, same angle, same light. The direct page for the photo is [flickr.com/photos/sherrent/3143192750](https://www.flickr.com/photos/sherrent/3143192750/).

![](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/iamge%209.png)

The Flickr metadata shows the photo was taken on **March 22, 2006**.

**Part 5 answer: `22/03/2006`**

---

**Part 6 — license plate**

This was the last remaining piece and the one that took the longest with the least obvious path.

Florida license plate data is not publicly searchable through any normal channel.
There was no plate visible anywhere in the photos we had, not in the bridge photo, not in the rest of the Flickr gallery, not in any image on Instagram or Facebook.

We went back through the Flickr account more carefully, looking at every photo in the gallery to see if any of them had a car or a plate visible in the frame. There were a few outdoor shots, some interior photos, nothing with a readable plate.

At this point we started thinking about where a license plate could realistically show up in connection to a person online. Insurance documents, car listings, rental agreements, all private. But review platforms connected to car-based businesses were worth checking.

We searched for "Arnold Sherr" across review sites and also checked [Ratelivo](https://ratelivo.com/businesses/tommys-express.com) which aggregates reviews from multiple platforms. He had not left many reviews publicly but we found one on **Trustpilot** for **Tommy's Express** which is a car wash chain. Tommy's Express is relevant because their entire service model is built around automatic license plate recognition. Customers register their plate with the app and the system identifies them when they pull into the wash bay.

![](https://raw.githubusercontent.com/wal-z1/ctf-writeups/main/.gitbook/assets/image2.png)

The review is dated January 5, 2026 and marked as "Unprompted review." Arnold is complaining about being overcharged on his membership for years, going back and forth with a representative named Dana for two months without resolution and the price being raised without proper notice. The review ends with:

> "Arnold Sherr, License Plate ZZ9"

Trustpilot link: [Tommy's Express reviews](https://www.trustpilot.com/review/tommys-express.com?stars=1)

**Part 6 answer: `ZZ9`**

---

**Flag**

```
mctf{LARGO-1995_DANIELLE-GRAEF_83_THE-ZOHAN_22/03/2006_ZZ9}
```

---

**References**

- [Get A Job Manual — Google Maps](https://www.google.com/maps/place/Get+A+Job+Manual/@48.6170322,-153.3785226,15016180m/data=!3m1!1e3!4m18!1m9!3m8!1s0x88c2f122c487f231:0x70a74d6ae55c63b2!2sGet+A+Job+Manual!8m2!3d46.423669!4d-129.9427086!9m1!1b1!16s%2Fg%2F11h4w9sk47!3m7!1s0x88c2f122c487f231:0x70a74d6ae55c63b2!8m2!3d46.423669!4d-129.9427086!9m1!1b1!16s%2Fg%2F11h4w9sk47?entry=ttu&g_ep=EgoyMDI2MDQyOC4wIKXMDSoASAFQAw%3D%3D)
- [theresumestore.com](https://theresumestore.com/)
- [The Resume Store — LinkedIn](https://www.linkedin.com/in/theresumestore/)
- [The Resume Store — Twitter](https://twitter.com/StoreResume)
- [The Resume Store — Instagram](https://www.instagram.com/theresumestore/)
- [The Resume Store — Facebook](https://web.facebook.com/TRSResumes/)
- [The Resume Store — WordPress blog](https://theresumestore.wordpress.com/)
- [Arnold Sherr — Flickr account](https://www.flickr.com/photos/sherrent/)
- [Arnold Sherr — Flickr bridge photo](https://www.flickr.com/photos/sherrent/3143192750/)
- [Arnold Sherr — Blogger profile](https://www.blogger.com/profile/10356161715111330278)
- [Arnold Sherr — Truthfinder listing](https://www.truthfinder.com/report-review/?utm_source=WXAB&traffic%5Bsource%5D=WXAB&utm_medium=affiliate&traffic%5Bmedium%5D=affiliate&utm_campaign=FL&traffic%5Bcampaign%5D=record%3AFL&utm_term=more_about&traffic%5Bterm%5D=more_about&utm_content=driving-records&traffic%5Bcontent%5D=driving-records&s1=FL&s2=record&s3=more_about&s4=driving-records&s5=&traffic%5Bfunnel%5D=bg&ck_rsid=4006457186&firstName=Arnold&lastName=Sherr&city=Largo&state=FL&age=83&tcg_id=1151362b-7369-4253-b275-998142e772d5&transaction_id=f1a9089a-5a60-465e-aeec-26cfc2799c96&index=1&bestResult=false&intent=curiosity)
- [Cumberland Farms — encyclopedia.com](https://www.encyclopedia.com/social-sciences-and-law/economics-business-and-labor/businesses-and-occupations/cumberland-farms-inc)
- [Danielle Graef — LinkedIn](https://www.linkedin.com/in/danielle-graef-financial-csr/)
- [Tommy's Express — Ratelivo](https://ratelivo.com/businesses/tommys-express.com)
- [Tommy's Express — Trustpilot](https://www.trustpilot.com/review/tommys-express.com?stars=1)
