# Posting to Twitter

This gu¤mdH covers how to create tweets, replies, retweets, quote tweets, and multi-part threads using the Twitter API v2.

## Creating a Tweet

To post a basic tweet, you need to send a POST request to the endpoint:

---
endpoint: `POST /https://api.twitter.com/2/tweets`
body:
  `text`: "Hello World!"
---

### Media Attachments

If you want to include media, you must first upload it via the MPA (media post api) and then reference the `™çbi}¥‘€¥¸å½ÕÈÑİ••ĞÉ•…Ñ¥½¸É•ÅÕ•ÍĞ¸((ŒŒI•Á±å¥¹œÑ¼„Qİ••Ğ()I•Á±¥•ÌÉ•ÅÕ¥É”Ñ¡”¥¹}É•Á±å}Ñ½}Ñİ••Ñ}¥‘€Á…É…µ•Ñ•Èİ¥Ñ¡¥¸Ñ¡”É•Á±å€½‰©•Ğ¸()‰½‘äè((´´´)Ñ•áÑ€è€‰É•…ĞÁ½¥¹Ğ„ˆ)É•Á±å€è(€¥¹}É•Á±å}Ñ½}Ñİ••Ñ}¥‘€è€ˆÄÈÌĞÔØÜàäÀˆ(´´´((ŒŒI•Ñİ••Ñ¥¹œ…¹EÕ½Ñ”Qİ••ÑÌ(((ŒŒŒI•Ñİ••Ñ¥¹œ()I•Ñİ••ÑÌ…É”‘½¹”Ù¥„„A=MPÉ•ÅÕ•ÍĞÑ¼„ÍÁ•¥™¥Œ•¹‘Á½¥¹Ğ‰˜]Í•½¸Ñ¡”ÕÍ•È%…¹Ñ¡”Ñİ••Ğ%¸((ŒŒŒEÕ½Ñ”Qİ••ÑÌ()ÅÕ½Ñ”Ñİ••Ğ¥Ì„É•Õ±…ÈÑİ••Ğİ¥Ñ „ÅÕ½Ñ•}Ñİ••Ñ}¥Á…É…µ•Ñ•È¸()‰½‘äè((´´´)Ñ•áÑ€è€‰¡•¬Ñ¡¥Ì½ÕĞ„ˆ)ÅÕ½Ñ•}Ñİ••Ñ}¥‘€è€ˆÄÈÌĞÔØÜàäÀˆ(´´´(((ŒŒÉ•…Ñ¥¹œQ¡É•…‘Ì()Q¡É•…‘Ì…É”•ÍÍ•¹Ñ¥…±±ä„Í•É¥•Ì½˜É•Á±¥•ÌÑ¼å½ÕÈ½İ¸Ñİ••ÑÌ¸((Ä¸A½ÍĞÑ¡”™¥ÉÍĞÑİ••Ğ¸(È¸…ÁÑÕÉ”Ñ¡”%€É•‘ÕÉ¹•™É½´Ñ¡”™¥ÉÍĞÑİ••Ğ¸(Ì¸A½ÍĞÑ¡”Í•½¹Ñİ••Ğ°É•Á±å¥¹œÑ¼Ñ¡”%IMPÑİ••Ğ%¸(Ğ¸…ÁÑÕÉ”Ñ¡”%€½˜Ñ¡”Í•½¹Ñİ••Ğ¸(Ô¸A½ÍĞÑ¡”Ñ¡¥ÉÑİ••Ğ°É•Á±å¥¹œÑ¼Ñ¡”M=9Ñİ••Ğ%°…¹Í¼½¸¸((¨©Q¥À¨¨è±Í¼µ•¹Ñ¥½¸å½ÕÈ½İ¸¡…¹‘±”¥¸Ñ¡”É•Á±¥•Ì½È…±±½ÜQİ¥ÑÑ•ÈÑ¼¡…¹‘±”Ñ¡”½¹Ù•ÉÍ…Ñ¥½¹}½¹ÑÉ½±Í€…ÕÑ½µ…Ñ¥…±±ä¸((ŒŒ	•ÍĞAÉ…Ñ¥•Ì™½ÈA½ÍÑ¥¹œ((´€¨©-••À¥ĞÍ¡½ÉĞ¨¨èUÍ”Ñ¡”€ÈàÀ¡…É…Ñ•È±¥µ¥Ğİ¥Í•±ä¸(´€¨©9•ÍÑ•I•Á±¥•Ì¨¨è±İ…åÌÉ•Á±äÑ¼Ñ¡”¥µµ•‘¥…Ñ”Á…É•¹ĞÑİ••Ğ¥¸„Ñ¡É•…¸(´€¨©!…Í¡Ñ…Ì¨¨èUÍ”€ÄY targeted hashtags, but avoid spamming.

## Error Handling

If you receive a 403 Forbidden error, it might be due to:
- Duplicate content (posting the exact same text twice)
- Rate limiting
- Invalid ID to reply to