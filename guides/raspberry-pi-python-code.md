---
description: >-
  Python code which collects Arduino data by bluetooth and sends it to REST API
  using APIGateway
---

# Raspberry Pi Data Sender

```python
from bluetooth import *  # sudo apt-get install bluetooth
import datetime
import json
import time
import requests  # pip install requests

# for amazon auth
import sys
import os
import base64
import hashlib
import hmac

URL = 'https://idFromAPIGateway.execute-api.ap-northeast-2.amazonaws.com/prod'

# Read AWS access key from env. variables or configuration file. Best Practice is NOT
# to embed credentials in code
API_KEY = os.environ.get('AWS_API_KEY')
ACCESS_KEY = os.environ.get('AWS_ACCESS_KEY_ID')
SECRET_KEY = os.environ.get('AWS_SECRET_ACCESS_KEY')

# Key derivation functions. See:
# http://docs.aws.amazon.com/general/latest/gr/signature-v4-examples.html#signature-v4-examples-python
def sign(key, msg):
    return hmac.new(key, msg.encode('utf-8'), hashlib.sha256).digest()


def getSignatureKey(key, dateStamp, regionName, serviceName):
    kDate = sign(('AWS4' + key).encode('utf-8'), dateStamp)
    kRegion = sign(kDate, regionName)
    kService = sign(kRegion, serviceName)
    kSigning = sign(kService, 'aws4_request')
    return kSigning


def awsRequest(requestBody):
    # requested values
    method = 'POST'
    service = 'execute-api'
    host = 'apigateway.amazonaws.com'
    region = 'ap-northeast-2'
    endpoint = 'https://hmwzfc5hga.execute-api.ap-northeast-2.amazonaws.com/prod/dust'
    content_type = 'application/json'
    # POST requests use a content type header. For DynamoDB,
    # the content is JSON.
    # content_type = 'application/x-amz-json-1.0'

    if ACCESS_KEY is None or SECRET_KEY is None:
        print('No access key is available.')
        sys.exit()

    # Create a date for headers and the credential string
    t = datetime.datetime.utcnow()
    # Format date as YYYYMMDD'T'HHMMSS'Z'
    amz_date = t.strftime('%Y%m%dT%H%M%SZ')
    # Date w/o time, used in credential scope
    date_stamp = t.strftime('%Y%m%d')

    # ************* TASK 1: CREATE A CANONICAL REQUEST *************
    # http://docs.aws.amazon.com/general/latest/gr/sigv4-create-canonical-request.html

    # Step 1 is to define the verb (GET, POST, etc.)--already done.

    # Step 2: Create canonical URI--the part of the URI from domain to query
    # string (use '/' if no path)
    canonical_uri = '/'

    # Step 3: Create the canonical query string. In this example, request
    # parameters are passed in the body of the request and the query string
    # is blank.
    canonical_querystring = ''

    # Step 4: Create the canonical headers. Header names must be trimmed
    # and lowercase, and sorted in code point order from low to high.
    # Note that there is a trailing \n.
    canonical_headers = 'host:' + host + '\n' + 'x-amz-date:' + amz_date + '\n'
    # canonical_headers = 'content-type:' + content_type + '\n' + 'host:' + host + '\n' + 'x-amz-date:' + amz_date + '\n'

    # Step 5: Create the list of signed headers. This lists the headers
    # in the canonical_headers list, delimited with ";" and in alpha order.
    # Note: The request can include any headers; canonical_headers and
    # signed_headers include those that you want to be included in the
    # hash of the request. "Host" and "x-amz-date" are always required.
    # For DynamoDB, content-type and x-amz-target are also required.
    signed_headers = 'host;x-amz-date;'

    # Step 6: Create payload hash. In this example, the payload (body of
    # the request) contains the request parameters.
    payload_hash = hashlib.sha256(requestBody.encode('utf-8')).hexdigest()

    # Step 7: Combine elements to create canonical request
    canonical_request = method + '\n' + canonical_uri + '\n' + canonical_querystring + \
        '\n' + canonical_headers + '\n' + signed_headers + '\n' + payload_hash

    # ************* TASK 2: CREATE THE STRING TO SIGN*************
    # Match the algorithm to the hashing algorithm you use, either SHA-1 or
    # SHA-256 (recommended)

    # strCredential = 'AKIAJWA2SJKXKVPF3P5A/20181026/ap-northeast-2/apigateway/aws4_request'

    algorithm = 'AWS4-HMAC-SHA256'
    credential_scope = date_stamp + '/' + region + \
        '/' + service + '/' + 'aws4_request'
    # credential_scope = date_stamp + '/' + region + '/' + service + '/' + 'aws4_request'
    string_to_sign = algorithm + '\n' + amz_date + '\n' + credential_scope + \
        '\n' + hashlib.sha256(canonical_request.encode('utf-8')).hexdigest()

    # ************* TASK 3: CALCULATE THE SIGNATURE *************
    # Create the signing key using the function defined above.
    signing_key = getSignatureKey(SECRET_KEY, date_stamp, region, service)

    # Sign the string_to_sign using the signing_key
    signature = hmac.new(signing_key, (string_to_sign).encode(
        'utf-8'), hashlib.sha256).hexdigest()

    # ************* TASK 4: ADD SIGNING INFORMATION TO THE REQUEST *************
    # Put the signature information in a header named Authorization.
    authorization_header = algorithm + ' ' + 'Credential=' + ACCESS_KEY + '/' + \
        credential_scope + ', ' + 'SignedHeaders=' + \
        signed_headers + ', ' + 'Signature=' + signature

    # For DynamoDB, the request can include any headers, but MUST include "host", "x-amz-date",
    # "x-amz-target", "content-type", and "Authorization". Except for the authorization
    # header, the headers must be included in the canonical_headers and signed_headers values, as
    # noted earlier. Order here is not significant.
    # # Python note: The 'host' header is added automatically by the Python 'requests' library.
    headers = {'Content-Type': content_type,
               'X-Amz-Date': amz_date,
               'X-Api-key': API_KEY,
               'Authorization': authorization_header}

    # ************* SEND THE REQUEST *************
    print('\nBEGIN REQUEST++++++++++++++++++++++++++++++++++++')
    print('Request URL = ' + endpoint)

    r = requests.post(endpoint, data=requestBody, headers=headers)

    print('\nRESPONSE++++++++++++++++++++++++++++++++++++')
    print('Response code: %d\n' % r.status_code)
    print(r.text)


###### EXECUTION CODES ######
print('----------------------- AWS Data Sender -----------------------')
print('')
print('  Ver. 1.0.')
print('  Made by 100kimch@naver.com')
print('')
print('  Get bluetooth data from Arduino and send to AWS APIGateway.')
print('')
print('---------------------------------------------------------------')

# bd_addr = "00:18:E4:35:48:0F"
bd_addr = {
    1: "00:18:E4:35:4A:D7",
    2: "00:18:E4:35:48:09",
    # 3: "00:18:E4:35:48:0F"
}
connected = []
# port = 1
period = 7.0
sock = {}

# print('[Msg] Connection Seccess\n[Msg] Data will be received in ' +
#       str(period)+' Seconds.')
while 1:
    for i in bd_addr:
        try:
            sock = BluetoothSocket(RFCOMM)
            sock.connect((bd_addr[i], 1))
            connected.append(i)
            print('[Msg] Connection Success: (' + str(i) + ')', bd_addr[i])
        except BluetoothError as err:
            print('[Msg] Connection failed: (' + str(i) + ')' + bd_addr[i], err)
            continue
            
        print('[Msg] Connection Success except error-occured devices.')

        time.sleep(period/2)

        try:
            sockRecv = sock.recv(100)
        except BluetoothError as err:
            print('[Msg] could not received data.')
            continue
        # print('[Dev] data:', sockRecv)
        data = sockRecv.decode('utf-8', 'ignore')
        print('[Msg]' + str(data))
        try:
            check1 = data.index("{")
            check2 = data[check1:].index("}") + check1
            check = True
        except ValueError:
            check = False
        if check:
            data = data[check1+1:check2].split(':')
            jsonData = {
                'time': str(datetime.datetime.now()),
                'id': data[0],
                'dust': data[1],
                'temp': data[2],
                'humid': data[3]
            }
            print('[Msg] jsonData: ', jsonData)
            awsRequest(json.dumps(jsonData))

            # uploadRate can be set in Arduino Code.
            # period = float(data[4])
            # print('[Msg] Sending rate changed to ' + data[4])
            # time.sleep(period)
        else:
            print('[Msg] Failed to get data.')
        sock.close()
        time.sleep(period/2)

        # print('[Msg] Socket Closed.')
#############################


```

