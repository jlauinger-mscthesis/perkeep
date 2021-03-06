# Blob Upload Resume

Optional upload protocol extension:

Blobs can be large, devices (e.g. mobile phones) can have slow
uploads, or both.  Thus, it's nice to have an upload resume mechanism.

In a stat response, a server can return a JSON key
"alreadyHavePartially" with similar format to the spec-required "stat"
array.  Instead of just "blobRef" and "size", though, there's a
continuation key and blobref of the part that server already has:

    ...
       "alreadyHavePartially": [
          {"blobRef": "sha1-abcdabcdabcdabcdabcdabcdabcdabcdabcdabcd",
           "size": 12312,
           "partBlobRef": "sha1-beefbeefbeefbeefbeefbeefbeefbeefbeefbeef"
           "resumeKey": "resume-sha1-abcdabcdabcdabcdabcdabcdabcdabcdabcdabcd-12312-server-chosen",
           }
       ],
    ...

If the client also supports this optional extension and parses the
"alreadyHavePartially" section, the client may resume their upload by:

1. verifying that digest of the client blob from byte 0 (incl) to
   "size" (exclusive) matches the server's provided "partBlobRef".
   (the server must use the same digest function).  if it doesn't,
   skip, and/or proceed to any other "alreadyHavePartially"
   blobref with the same "blobRef" value.  (the server may have
   multiple partial uploads in different states, and perhaps one
   is corrupt for various HTTP client failure reasons...)

2. do an upload like normal, but the name of the
   multipart/form-data body part should be whatever the server
   provided in the mandatory "resumeKey" value.  skip the first
   "size" bytes in your upload.

