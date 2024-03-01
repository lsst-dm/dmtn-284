#########################################
Signed URLs for Data Releases at the USDF
#########################################

.. abstract::

   Cybersecurity may make the initial design for issuing signed URLs for Data Release data at the USDF unworkable.  This DMTN describes the problem, several alternative solutions, and suggests a path forward.


Design
======

The USDF design expects to have released data placed on the Weka POSIX filesystem at SLAC.
As that filesystem must support staff and Data Release Production, it is expected to be highly reliable and performant.
By having released data be in the same location as the output of DRP, an extra copy step could be avoided.
In order to support RSP services, which require signed URLs to data, the design expected to use the `Weka S3 interface`_.

.. _Weka S3 interface: https://docs.weka.io/additional-protocols/s3/s3-examples-using-boto3#pre-signed-url-example


Problem
=======

In a conversation with Yee, he expressed the belief that SLAC Cybersecurity will want Weka, including its S3 interface, to remain on internal-only `RFC1918`_ IP addresses, not allowing direct connections from the Internet.
This means that we cannot give signed URLs to arbitrary science users to retrieve released data from the USDF Weka filesystem.

.. _RFC1918: https://www.rfc-editor.org/rfc/rfc1918


Alternatives
============

If we do not have the capability of making Weka S3 available on the Internet, there are a few alternatives:
- Deploy a proxy server to pass S3 through to the internal Weka S3.
This doesn't seem to accomplish much in terms of protecting Weka, as the proxy server still needs to be open to the entire Internet.

- Deploy our own S3 server at SLAC backed by Weka POSIX.  This seems like a maintenance headache, even if we got the code from elsewhere.

- Place a pass-through proxy server at Google Cloud and restrict connections to Weka S3 to specific Google Cloud IPs.  This may not be sufficient to satisfy Cybersecurity.  It may create substantial egress bandwidth from Google Cloud to end users, depending on the ratio of Cloud service vs. end user destinations.

- Deploy a server of a different protocol (like WebDAV or xrootd) backed by Weka POSIX.  It's not clear that these can be sufficiently authorized.  Alternative protocols may be more difficult for services and users to use.  (Translating these protocols to S3 could be even more difficult.)

- Copy data on request from SLAC to Google Cloud, then serve from the Cloud.  Again, there are potential egress problems, plus increased latency.  Our Google Cloud Platform partners have suggested this mode, however.

- Give up on Weka altogether and place the release data in Internet-accessible `Ceph S3`_.  This requires us to move a substantial portion of the storage we have purchased from the sdftier Ceph S3 (which serves as the disk-based backing store for the Weka POSIX filesystem) to s3dfrgw (or whatever replaces it as the S3DF S3 storage on the public Internet).  We should make sure that s3dfrgw is rearchitected to be much more highly available and reliable than the current implementation, along the lines of the Embargo Ceph S3 (or better).  Yee suggests that Weka S3, being simpler and commercially supported (and apparently based on the fairly well-known MinIO), may be more reliable than Ceph S3, but its performance and scalability are still unknown.

.. _Ceph S3: https://docs.ceph.com/en/latest/radosgw/

There are potential latency and usability impacts on Rubin staff if released data products are only in S3 and not visible in POSIX, but at least some staff have been living with this for LATISS and recent LSSTCam IR2 data without major complaints.


Conclusion
==========

The three pathways we should explore are thus:
- See if Weka S3 can be placed on the Internet.
- See if we can use a pass-through proxy server at the USDF to talk to Weka S3.
- See if we can develop a high-availability, high-reliability, high-performance, high-capacity, Internet-accessible Ceph S3 configuration to store Data Releases.
