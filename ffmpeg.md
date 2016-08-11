## Some crib notes for using FFMPEG

### Download video from LiveIL
ffmpeg -i 'http://c.brightcove.com/services/mobile/streaming/index/master.m3u8?videoId=5050833136001' -c copy -bsf:a aac_adtstoasc 'Amazing Race S05E19.mp4'
