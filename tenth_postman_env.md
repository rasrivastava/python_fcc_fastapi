## Postman environment

An environment is a set of variables that allows you to switch the context of your requests.
<img width="1435" alt="Screenshot 2021-11-20 at 13 24 13" src="https://user-images.githubusercontent.com/11652564/142719021-80bff89a-b60e-4cdf-b25c-66297f65feab.png">

<img width="1043" alt="Screenshot 2021-11-20 at 13 24 26" src="https://user-images.githubusercontent.com/11652564/142719029-22aac31c-dda6-4bab-92b9-f8a618075adb.png">

- fails to authorize as we have not added the token
<img width="1005" alt="Screenshot 2021-11-20 at 13 25 03" src="https://user-images.githubusercontent.com/11652564/142719043-11189db1-4671-4865-8f4a-4a9c17e0a61a.png">

- adding token

<img width="1021" alt="Screenshot 2021-11-20 at 13 25 36" src="https://user-images.githubusercontent.com/11652564/142719055-be428d17-2355-4129-96aa-0445fa9ae3d7.png">


- create a new token
  - then go the `Tests`
  - Add `pm.environment.set("JWT", pm.response.json().access_token);`
  - now, we will go to any of the api for ex: `{{URL}}posts` (posts) and in `authorisation` in `Token` section add the env variable **{{JWT}}**


<img width="1023" alt="Screenshot 2021-11-20 at 13 28 37" src="https://user-images.githubusercontent.com/11652564/142719114-0a2090d5-24d8-436b-9ddc-808f817d41ba.png">


<img width="1022" alt="Screenshot 2021-11-20 at 13 28 55" src="https://user-images.githubusercontent.com/11652564/142719127-d6d8cc29-2774-43f1-ba95-9255b9bcde24.png">

<img width="1033" alt="Screenshot 2021-11-20 at 13 31 24" src="https://user-images.githubusercontent.com/11652564/142719177-fe75702b-849d-410e-9d79-cc0ac101257f.png">

<img width="1029" alt="Screenshot 2021-11-20 at 13 32 07" src="https://user-images.githubusercontent.com/11652564/142719195-939bc722-ac54-4ab6-ba62-916debf2ed73.png">

<img width="1020" alt="Screenshot 2021-11-20 at 13 34 27" src="https://user-images.githubusercontent.com/11652564/142719246-3c1e622b-869b-4d37-b8f6-606a713ff955.png">

