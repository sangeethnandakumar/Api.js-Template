# Api.js Common API helper framework (Using Axios + MSAL)
Revealing Module Pattern-based generic 'Api.js' template (using Axios and MSAL)

> For use in JavaScript projects (React, Angular etc..)

### Steps to use
- Install Axios (`npm i axios`)
- Replace `baseURL` below
- Replace `scopes` below
- Ready to use

> Based on `bool` value we pass in `isFormData`, Api.js will either put data as Key-Value FormData or as Raw JSON body

> If you didn't pass `msalInstance`, Api.js assumes authorization is not required for that endpoint and never try token accusation

## Api.js

```js
import axios from 'axios';

const baseURL = 'https://localhost:7120';
const scopes = [
    'openid',
    'offline_access',
    'profile',
    'email',
    'https://xxxx.onmicrosoft.com/xxxxxxxxxxxxx/Files.Read'
];

const Api = (function () {
    const axiosInstance = axios.create({
        baseURL: baseURL,
    });

    const tryRelogin = (msalInstance) => {
        if (msalInstance) {
            msalInstance.logoutPopup();
            setTimeout(() => msalInstance.loginPopup(), 2000);
        }
    }

    const fetchToken = async (msalInstance) => {
        if (!msalInstance) return null;

        try {
            const token = await msalInstance.acquireTokenSilent({
                scopes: scopes,
            });
            return token.accessToken;
        } catch (error) {
            if (msalInstance) {
                tryRelogin(msalInstance);
            }
        }
    }

    const handleResponse = (response, onSuccess, onFailure, msalInstance) => {
        if (response.status === 200) {
            onSuccess(response.data);
        } else if (response.status === 401 && msalInstance) {
            tryRelogin(msalInstance);
        } else {
            onFailure(response);
        }
    }

    const makeRequest = async (method, url, data, isFormData, onSuccess, onFailure, msalInstance) => {
        try {
            const token = await fetchToken(msalInstance);
            const headers = isFormData ? { 'Content-Type': 'multipart/form-data' } : {};

            if (token) {
                headers['Authorization'] = `Bearer ${token}`;
            }

            const response = await method(url, data, { headers });
            handleResponse(response, onSuccess, onFailure, msalInstance);
        } catch (error) {
            onFailure(error);
        }
    }

    return {
        GET: (url, params, onSuccess, onFailure, msalInstance) => makeRequest(axiosInstance.get, url, { params }, false, onSuccess, onFailure, msalInstance),
        POST: (url, data, isFormData, onSuccess, onFailure, msalInstance) => makeRequest(axiosInstance.post, url, data, isFormData, onSuccess, onFailure, msalInstance),
        PUT: (url, data, isFormData, onSuccess, onFailure, msalInstance) => makeRequest(axiosInstance.put, url, data, isFormData, onSuccess, onFailure, msalInstance),
        PATCH: (url, data, isFormData, onSuccess, onFailure, msalInstance) => makeRequest(axiosInstance.patch, url, data, isFormData, onSuccess, onFailure, msalInstance),
        DELETE: (url, onSuccess, onFailure, msalInstance) => makeRequest(axiosInstance.delete, url, null, false, onSuccess, onFailure, msalInstance),
    };
})();

export default Api;
```

## Calling From a Component
This is how we can call from a component

```js
import Api from '../Api';
import { useMsal } from '@azure/msal-react';


//GET
const retrieveData = () => {
    Api.GET('Subscriptions/GetData',
        { param1: 'value1', param2: 'value2' },
        data => {
            alert(JSON.stringify(data));
        },
        err => {
            alert('failure');
        },
        instance);
}

//POST
const postData = () => {
    Api.POST('Subscriptions/SubmitData',
        {
            Field1: 'Value1',
            Field2: 'Value2'
        },
        true, // Use form data
        data => {
            alert(JSON.stringify(data));
        },
        err => {
            alert('failure');
        },
        instance);
}

//PUT
const updateData = () => {
    Api.PUT('Subscriptions/UpdateData',
        {
            Field1: 'UpdatedValue1',
            Field2: 'UpdatedValue2'
        },
        false, // Use JSON data
        data => {
            alert(JSON.stringify(data));
        },
        err => {
            alert('failure');
        },
        instance);
}

//PATCH
const patchData = () => {
    Api.PATCH('Subscriptions/PatchData',
        {
            Field1: 'PatchedValue1',
            Field2: 'PatchedValue2'
        },
        false, // Use JSON data
        data => {
            alert(JSON.stringify(data));
        },
        err => {
            alert('failure');
        },
        instance);
}

//DELETE
const deleteData = () => {
    Api.DELETE('Subscriptions/DeleteData',
        data => {
            alert('Data deleted successfully');
        },
        err => {
            alert('failure');
        },
        instance);
}
```
