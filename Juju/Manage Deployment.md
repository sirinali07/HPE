 ### Task 1 : scaling an Apache2 application deployed with Juju

 Check Current Status:
 ```
juju status apache2
```
Show detailed information for a specific unit:
```
juju show-unit apache2/0
```
Add additional units to the Apache2 application:
```
juju add-unit apache2 -n 2
```
Check the status again to verify the new units:
```
juju status apache2
```
Remove Specific Units from Apache2:
```
juju remove-unit apache2/1
```
```
juju remove-unit apache2/2
```
Check the status again to verify the removal:
```
juju status apache2
```
 ### Task 2 : Set Constraints on the Apache2 Application
 Set constraints for the Apache2 application to use t2.medium instances:
```
juju set-constraints apache2 instance-type=t2.medium
```
Add 2 additional units to the Apache2 application with the new constraints:
```
juju add-unit apache2 -n 2
```
`Note:` This command scales the Apache2 application by adding two more units, which will be provisioned using the `t2.medium` instance type as per the set constraints.

Verify  the current resource constraints set:
```
juju constraints apache2
```
Check the status again to verify the new units with the new constraints:
```
juju status apache2
```







