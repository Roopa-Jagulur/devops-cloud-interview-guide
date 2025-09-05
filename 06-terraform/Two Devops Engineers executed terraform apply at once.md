Here concept called statefile locking will only allow one engineer to work on statefile at a time for applying /modifying resources. 
the statefile which is stored in the backend (s3/blob/gcp) impliment locking of the statefile.



-Both of their requests head to S3 parallely and typically there is a race condition between the DevOps incidents. Due to request latency between two request, one which reached S3 is processed first, locking statefile with the first requester name (error is shown to another engineer) and releases once request is processed fully.

next request and so on.

- Say there is no latency - request reaches same time then cloud provider, remote backend (S3, blob, gcp) will have race condition mechanism to handle those kind of requests to handle one request and error displayed for another engineer who tries to run same request.

next request and so on.
