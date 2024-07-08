

1.  you can make a file `loading.tsx` in the folder where a `page.tsx` is present with the export as an `async function`, and has an async call before fetching. the example is given below. this is a server component, loads the data and then html is returned
2. 
   
   ```tsx
// this is page.tsx

import axios from "axios";

async function getUserData () {

const response = await axios get ("https://week-13-offline.kirattechnologies.workers.dev/api/v1/user/

details")

return response.data;

// async component

Sport default async function Home() {

const userDetails = await getUserData();

return (

<div className="flex flex-col justify-center h-screen">

<div className="flex justify-center">

<div className="border p-8 rounded">

<div>

Name: {userDetails?. name}

</div>
{userDetails?. email)
</div>
</div>
</div>

);
```



