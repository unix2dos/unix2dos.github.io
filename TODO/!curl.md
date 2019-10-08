http://www.ruanyifeng.com/blog/2011/09/curl.html









-X  POST DELETE

-H   header

-d --data  --data-urlencode   Data

curl 127.0.0.1:8866/api/v1/UpdateTranscodeDetails  -X POST --data-urlencode  'id=b6a1ec7a-0cf5-487c-972d-16adbe4a3e49&key=YIN&value=-vf scale%3d-2:360 -c:v libx264 -pix_fmt yuv420p'