## 🔐Password expired or  Using a Default Password Redirect to Change Password Page:

In some cases, a user's password may be reset to a known default (e.g., 'Welcome123') or may have expired. To force these users to change their password immediately after login, follow these steps.

---

### 🔧 Implementation Steps

#### 1. Add a Flag or Logic to Detect Default Password

There are two common ways to check for default password usage:

### 2. Create a "Change Password" Page

Create a page in your APEX app (e.g., Page **1001**) where users can change their password.

---

**Store a "Password Reset" Flag in the User Table**

Add two columns to your user table (if not already there):

```sql
ALTER TABLE ACL_USER ADD password_reset_required CHAR(1);
ALTER TABLE ACL_USER ADD expiry_date DATE;

```
### 3. Create a Function to Check Password Expiry or Default Password

Create a function that checks if the user’s password is expired or Default Password.

---

```sql function
CREATE OR REPLACE FUNCTION fun_check_pwd_status (
    p_username IN VARCHAR2
)
RETURN VARCHAR2
IS
    l_pwd_expired_flag       CHAR(1);
    l_default_password_flag  CHAR(1);
    l_user_exists            NUMBER;
BEGIN
    -- First check if user exists
    SELECT COUNT(*)
    INTO l_user_exists
    FROM ACL_USER
    WHERE EMAIL_ID = p_username;
    
    -- If user doesn't exist, return appropriate status
    IF l_user_exists = 0 THEN
        RETURN 'USER_NOT_FOUND';
    END IF;

    SELECT CASE
             WHEN expiry_date < SYSDATE THEN 'Y'
             ELSE 'N'
           END,
           CASE
             WHEN password_reset_required = 'Y' THEN 'Y'
             ELSE 'N'
           END
      INTO l_pwd_expired_flag,
           l_default_password_flag
      FROM ACL_USER
     WHERE EMAIL_ID = p_username;

    -- Return status instead of redirecting (functions shouldn't perform redirects)
    IF l_pwd_expired_flag = 'Y' AND l_default_password_flag = 'Y' THEN
        RETURN 'BOTH_EXPIRED_AND_RESET_REQUIRED';
    ELSIF l_pwd_expired_flag = 'Y' THEN
        RETURN 'PASSWORD_EXPIRED';
    ELSIF l_default_password_flag = 'Y' THEN
        RETURN 'PASSWORD_RESET_REQUIRED';
    ELSE
        RETURN 'PASSWORD_VALID';
    END IF;
    
EXCEPTION
    WHEN NO_DATA_FOUND THEN
        RETURN 'USER_NOT_FOUND';
    WHEN TOO_MANY_ROWS THEN
        RETURN 'MULTIPLE_USERS_FOUND';
    WHEN OTHERS THEN
        RETURN 'ERROR: ' || SQLERRM;
END fun_check_pwd_status;
/
```

### 4. Add Redirect Logic After Login

On your Home Page (or your landing page), add a Before Header Process to redirect expired users or Default Password (Example usage in your application logic):

--- 

``` Redirect to Change page (Example usage in your application logic):

DECLARE
    l_status VARCHAR2(50);
BEGIN
    l_status := fun_check_pwd_status(lower(:APP_USER));
    
    IF l_status IN ('PASSWORD_EXPIRED', 'PASSWORD_RESET_REQUIRED', 'BOTH_EXPIRED_AND_RESET_REQUIRED') THEN
        -- Perform redirect in your application code, not in the function
        APEX_UTIL.REDIRECT_URL('f?p=&APP_ID.:38:&SESSION.::NO::');
    END IF;
END;
```

 # Thank you
 ## Sanjay Sikder

 You can connect with me on [LinkedIn](https://www.linkedin.com/in/sanjay-sikder/)!
