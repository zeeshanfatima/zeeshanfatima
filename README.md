import time
from urllib.error import URLError, HTTPError
import hashlib
import sys
from urllib.request import urlopen, Request
from playsound import playsound
from datetime import datetime

# Setting the URL you want to monitor
url = Request(
  ' https://service2.diplo.de/rktermin/extern/appointment_showMonth.do?locationCode=kara&realmId=967&categoryId=2801,
 #   'https://www.youtube.com/',
    headers={'User-Agent': 'Mozilla/5.0'}
)

def check_website():
    try:
        response = urlopen(url).read()
        return hashlib.sha224(response).hexdigest()
    except HTTPError as e:
        print(f"HTTP Error: {e.code} - {e.reason}")
        raise
    except URLError as e:
        if hasattr(e, 'reason'):
            print(f"Failed to reach server: {e.reason}")
        elif hasattr(e, 'code'):
            print(f"Server couldn't fulfill the request: {e.code}")
        raise
    except Exception as e:
        print(f"Unexpected error: {e}")
        raise

# To create the initial hash
try:
    currentHash = check_website()
except Exception as e:
    print(f"Initial Error: {e}")
    sys.exit(1)

print("running")

while True:
    try:
        # Wait for 5 seconds
        time.sleep(5)

        # Perform the GET request and create a new hash
        newHash = check_website()

        # Get the current time
        now = datetime.now()
        formatted_time = now.strftime("%Y-%m-%d %H:%M:%S")

        # Check if new hash is same as the previous hash
        if newHash == currentHash:
            print(f"{formatted_time} Appointment NOT Opened")
        else:
            print(f"{formatted_time} ******* Appointment Opened ******")

            # Play the alarm sound
            playsound('alarm.mp3')

            # Update the current hash with the new hash
            currentHash = newHash

    except (URLError, HTTPError) as e:
        # Get the current time
        now = datetime.now()
        formatted_time = now.strftime("%Y-%m-%d %H:%M:%S")

        # Handle URL and HTTP errors specifically
        print(f"{formatted_time} Error: {e}")
        continue

    except Exception as e:
        # Get the current time
        now = datetime.now()
        formatted_time = now.strftime("%Y-%m-%d %H:%M:%S")

        # Handle any other exceptions
        print(f"{formatted_time} Unexpected error: {e}")
        sys.exit(1)

