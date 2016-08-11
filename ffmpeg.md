## Some crib notes for using FFMPEG

### Download video from LiveIL
ffmpeg -i 'http://c.brightcove.com/services/mobile/streaming/index/master.m3u8?videoId=5066220689001' -c copy -bsf:a aac_adtstoasc 'Amazing Race S05E20.mp4'

ffmpeg -i 'http://c.brightcove.com/services/mobile/streaming/index/master.m3u8?videoId=5068618294001' -c copy -bsf:a aac_adtstoasc 'Amazing Race S05E21.mp4'

ffmpeg -i 'http://c.brightcove.com/services/mobile/streaming/index/master.m3u8?videoId=5074493476001' -c copy -bsf:a aac_adtstoasc 'Amazing Race S05E22.mp4'

ffmpeg -i 'http://c.brightcove.com/services/mobile/streaming/index/master.m3u8?videoId=5077291589001' -c copy -bsf:a aac_adtstoasc 'Amazing Race S05E23.mp4'

