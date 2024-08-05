### launch_args_example_pkg

#### Notes

##### 1. Omitted required argument

A declared argument without a `default_value=` is required. If not specified, ROS2 complains:
```
user:~/ros2_ws$ ros2 launch launch_args_example_pkg start_with_arguments_dummy.launch.py
[INFO] [launch]: All log files can be found below /home/user/.ros/log/2024-08-05-05-04-12-822562-1_xterm-6091
[INFO] [launch]: Default logging verbosity is set to INFO
Task exception was never retrieved
future: <Task finished name='Task-2' coro=<LaunchService._process_one_event() done, defined at /opt/ros/galactic/lib/python3.8/site-packages/launch/launch_service.py:226> exception=RuntimeError("Included launch description missing required argument 'important_msg' (description: 'no description given'), given: []")>
Traceback (most recent call last):
  File "/opt/ros/galactic/lib/python3.8/site-packages/launch/launch_service.py", line 228, in _process_one_event
    await self.__process_event(next_event)
  File "/opt/ros/galactic/lib/python3.8/site-packages/launch/launch_service.py", line 248, in __process_event
    visit_all_entities_and_collect_futures(entity, self.__context))
  File "/opt/ros/galactic/lib/python3.8/site-packages/launch/utilities/visit_all_entities_and_collect_futures_impl.py", line 45, in visit_all_entities_and_collect_futures
    futures_to_return += visit_all_entities_and_collect_futures(sub_entity, context)
  File "/opt/ros/galactic/lib/python3.8/site-packages/launch/utilities/visit_all_entities_and_collect_futures_impl.py", line 45, in visit_all_entities_and_collect_futures
    futures_to_return += visit_all_entities_and_collect_futures(sub_entity, context)
  File "/opt/ros/galactic/lib/python3.8/site-packages/launch/utilities/visit_all_entities_and_collect_futures_impl.py", line 38, in visit_all_entities_and_collect_futures
    sub_entities = entity.visit(context)
  File "/opt/ros/galactic/lib/python3.8/site-packages/launch/action.py", line 108, in visit
    return self.execute(context)
  File "/opt/ros/galactic/lib/python3.8/site-packages/launch/actions/include_launch_description.py", line 147, in execute
    raise RuntimeError(
RuntimeError: Included launch description missing required argument 'important_msg' (description: 'no description given'), given: []
```  

Providing a value to the required argument (the other has a default value):  
```
user:~/ros2_ws$ ros2 launch launch_args_example_pkg start_with_arguments_dummy.launch.py important_msg:="Here you go..."
[INFO] [launch]: All log files can be found below /home/user/.ros/log/2024-08-05-05-06-46-035581-1_xterm-6333
[INFO] [launch]: Default logging verbosity is set to INFO
[INFO] [launch.user]: hello world
[INFO] [launch.user]: Here you go...
```


