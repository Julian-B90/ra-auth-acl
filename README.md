# Access Control List for React Admin

Provide a small ACL framework to manage resources views access based on pre-defined roles.

## Getting Started

```bash
npm install --save-dev ra-auth-acl
# or
yarn add ra-auth-acl
```

## Usage

This library rely on the existing [React Admin permission management](https://marmelab.com/react-admin/Authorization.html) which allows to get permissions from an authenticated user.

1. Use `ResourceWithPermissions` instead of `Resource` of React Admin.

```jsx
import React from 'react';
import { Admin } from 'react-admin';

import authProvider from './authProvider';
import posts from './posts';
import comments from './comments';

render(
    <Admin authProvider={authProvider}>
        {permissions => [
            <ResourceWithPermissions name="posts" permissions={permissions} {...posts} />,
            <ResourceWithPermissions name="comments" permissions={permissions} {...comments} />,
        ]}
    </Admin>
);
```

2. On your [custom authentication provider](https://marmelab.com/react-admin/Authentication.html#configuring-the-auth-provider), you have to return your access control list when the framework ask for it.

```js
// authProvider.js

import { AUTH_GET_PERMISSIONS } from 'react-admin';

export default (type, params) => {
    // ...
    if (type === AUTH_GET_PERMISSIONS) {
        return Promise.resolve({
            posts: {
                // Use the resource `name` prop as identifier
                enabled: true,
                list: true,
                create: false,
                edit: false,
                show: true,
            },
            comments: {
                enabled: false, // This won't show the resource at all
                custom: true, // You can pass your own custom keys if needed
            },
        });
    }
};
```

The ACL works with five default permissions, all `false` by default:

- **enabled**: Display the resource or not.
- **list**: Display the list view of the resource
- **create**: Display the create view of the resource
- **edit**: Display the edit view of the resource
- **show**: Display the show view of the resource

You can go with your custom permissions and use the `hasAccess` helper to customize your resources:

```jsx
import { hasAccess } from 'ra-auth-acl';

export const UserList = ({ permissions, ...props }) => (
    <List {...props}>
        <Datagrid rowClick={hasAccess(permissions, 'users.edit') ? 'edit' : 'show'} expand={<UserEditEmbedded />}>
            <TextField source="id" />
            <TextField source="name" />
            {hasAccess(permissions, 'users.custom') && <TextField source="role" />}
        </Datagrid>
    </List>
);
```

## Dealing With Roles

Instead of building the permissions list each time, you can store the role on the local storage or in a JSON Web Token, and request your permissions list at runtime:

```js
// authProvider.js
import { AUTH_GET_PERMISSIONS } from 'react-admin';
import { buildFullAccessFor } from 'ra-auth-acl';

const permissions = {
    admin: {
        ...buildFullAccessFor(['posts', 'comments', 'users', 'tags']),
    },
    user: {
        ...buildFullAccessFor(['posts', 'comments', 'tags']),
    },
};

export default (type, params) => {
    if (type === AUTH_GET_PERMISSIONS) {
        const role = localStorage.getItem('role'); // Might be async request!
        return Promise.resolve(permissions[role]);
    }
};
```

## License

This library is licensed under the [MIT License](https://github.com/marmelab/comfygure/blob/master/LICENSE), sponsored and supported by [marmelab](http://marmelab.com).