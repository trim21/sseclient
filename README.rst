Server Side Events (SSE) client for Python
==========================================

A Python client for SSE event sources that seamlessly integrates with
``urllib3`` and ``requests``.

Installation
------------

.. code::

    $ pip install sseclient-py

Usage
-----

.. code:: python

    import sseclient

    def with_urllib3(url, headers):
        """Get a streaming response for the given event feed using urllib3."""
        import urllib3
        http = urllib3.PoolManager()
        return http.request('GET', url, preload_content=False, headers=headers)

    def with_requests(url, headers):
        """Get a streaming response for the given event feed using requests."""
        import requests
        return requests.get(url, stream=True, headers=headers)

    url = 'http://domain.com/events'
    headers = {'Accept': 'text/event-stream'}
    response = with_requests(url, headers)  # or with_urllib3(url, headers)
    client = sseclient.SSEClient(response)
    for event in client.events():
        # event.data contains the message payload
        # event.event contains the event type (default: 'message')
        # event.id contains the event ID (if provided by server)
        # event.retry contains the retry timeout (if provided by server)
        print(f'Event: {event.event}, Data: {event.data}')

Example with error handling and connection management:

.. code:: python

    import json
    import requests
    import sseclient

    url = 'http://domain.com/events'
    headers = {'Accept': 'text/event-stream'}

    response = requests.get(url, stream=True, headers=headers)
    client = sseclient.SSEClient(response)

    try:
        for event in client.events():
            # Handle different event types
            if event.event == 'message':
                # Parse JSON data if expected
                try:
                    data = json.loads(event.data)
                    print(f'Received message: {data}')
                except json.JSONDecodeError:
                    print(f'Received non-JSON message: {event.data}')
            elif event.event == 'ping':
                print('Received ping event')
            else:
                print(f'Received {event.event} event: {event.data}')
    except KeyboardInterrupt:
        print('Connection closed by user')
    finally:
        client.close()

Resources
=========

-  http://www.w3.org/TR/2009/WD-eventsource-20091029/
-  https://pypi.python.org/pypi/sseclient-py/
