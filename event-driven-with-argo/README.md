 
``` 
helm repo add argo https://argoproj.github.io/argo-helm
helm repo add localstack https://helm.localstack.cloud 
```


```python

from os import environ
from redis import Redis
from time import sleep

stream_key = environ.get("STREAM", "STREAM")
producer = environ.get("PRODUCER", "user-1")


def connect_to_redis():
    hostname = environ.get("REDIS_HOSTNAME", "redis-service") 
    port = environ.get("REDIS_PORT", 6379)

    return Redis(hostname, port, retry_on_timeout=True)


def send_event(redis_connection, aoi, reference):
    count = 0

    try:
        # TODO cloud events
        # un-map the "data" wrt app package parameters
        data = {
            "producer": producer,
            "href": reference,
        }
        resp = redis_connection.xadd(stream_key, data)
        print(resp)
        count += 1

    except ConnectionError as e:
        print(f"ERROR REDIS CONNECTION: {e}")


connection = connect_to_redis()

references = [
    "https://earth-search.aws.element84.com/v0/collections/sentinel-s2-l2a-cogs/items/S2A_10TFK_20210708_0_L2A",
    "https://earth-search.aws.element84.com/v0/collections/sentinel-s2-l2a-cogs/items/S2B_10TFK_20210713_0_L2A",
    "https://earth-search.aws.element84.com/v0/collections/sentinel-s2-l2a-cogs/items/S2A_10TFK_20210718_0_L2A",
    "https://earth-search.aws.element84.com/v0/collections/sentinel-s2-l2a-cogs/items/S2A_10TFK_20220524_0_L2A",
    "https://earth-search.aws.element84.com/v0/collections/sentinel-s2-l2a-cogs/items/S2A_10TFK_20220514_0_L2A",
    "https://earth-search.aws.element84.com/v0/collections/sentinel-s2-l2a-cogs/items/S2A_10TFK_20220504_0_L2A"
]

for reference in references:
    send_event(connection, aoi, reference)
    sleep(1)
```