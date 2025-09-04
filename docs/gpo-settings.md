# Group Policy Objects (GPO) Applied

1. **Password Policy**
   - Minimum length: 12
   - Complexity: Enabled
   - Max age: 90 days

2. **Drive Mapping**
   - H: → HR share (`\\server\hr`)
   - F: → Finance share (`\\server\finance`)

3. **USB Restriction**
   - Removable storage devices: Deny write access

4. **Login Restrictions**
   - HR users → Allowed only on HR machines
   - Finance users → Allowed only on Finance machines
