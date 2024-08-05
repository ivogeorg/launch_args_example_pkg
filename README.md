### launch_args_example_pkg

#### Notes

##### 1. Omitted required argument

1. A declared argument without a `default_value=` is required. If not specified, ROS2 complains:
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

2. Providing a value to the required argument (the other has a default value):  
```
user:~/ros2_ws$ ros2 launch launch_args_example_pkg start_with_arguments_dummy.launch.py important_msg:="Here you go..."
[INFO] [launch]: All log files can be found below /home/user/.ros/log/2024-08-05-05-06-46-035581-1_xterm-6333
[INFO] [launch]: Default logging verbosity is set to INFO
[INFO] [launch.user]: hello world
[INFO] [launch.user]: Here you go...
```

##### 2. Passing arguments directly to a class constructor

1. Passing an argument on the command line:
```
user:~/ros2_ws$ ros2 launch launch_args_example_pkg start_with_arguments.launch.py timer_period:=0.7
[INFO] [launch]: All log files can be found below /home/user/.ros/log/2024-08-05-05-31-27-309854-1_xterm-1279
[INFO] [launch]: Default logging verbosity is set to INFO
[INFO] [launch.user]: Tick
[INFO] [launch.user]: Tack
[INFO] [launch.user]: 0.7
[INFO] [arguments_examples_demo-1]: process started with pid [1289]
[arguments_examples_demo-1] [INFO] [1722835888.144361939] [dummy_arguments_example]: --- Tick ---
[arguments_examples_demo-1] [INFO] [1722835888.844457104] [dummy_arguments_example]: --- Tack ---
[arguments_examples_demo-1] [INFO] [1722835889.544370870] [dummy_arguments_example]: --- Tick ---
[arguments_examples_demo-1] [INFO] [1722835890.244369104] [dummy_arguments_example]: --- Tack ---
[arguments_examples_demo-1] [INFO] [1722835890.944454340] [dummy_arguments_example]: --- Tick ---
[arguments_examples_demo-1] [INFO] [1722835891.644365230] [dummy_arguments_example]: --- Tack ---
[arguments_examples_demo-1] [INFO] [1722835892.344362680] [dummy_arguments_example]: --- Tick ---
[arguments_examples_demo-1] [INFO] [1722835893.044366059] [dummy_arguments_example]: --- Tack ---
[arguments_examples_demo-1] [INFO] [1722835893.744358329] [dummy_arguments_example]: --- Tick ---
```

2. Passing the argument to the class constructor ([source](src/arguments_examples.cpp)):
```
int main(int argc, char *argv[]) {
  rclcpp::init(argc, argv);
  rclcpp::spin(std::make_shared<DummyArgumetsExample>(argc, argv));
  rclcpp::shutdown();
  return 0;
}
```

3. The constructor processes the passed arguments (notice that the first two have default values and didn't have to be passed explicitly)
```
  DummyArgumetsExample(int &argc, char **argv)
      : Node("dummy_arguments_example") {

    message1 = argv[2];
    message2 = argv[3];
    timer_period = argv[5];
    float tp1 = std::stof(timer_period);
    auto tp2 = std::chrono::duration<double>(tp1);

    timer_ = this->create_wall_timer(
        tp2, std::bind(&DummyArgumetsExample::timer_callback, this));

    timer_flag_ = true;
  }
```

4. The arguments were declared in the launch file:
```
import launch
from launch_ros.actions import Node

# How to use Example:
# ros2 launch execution_and_callbacks_examples start_with_arguments.launch.py timer_period:=0.5


def generate_launch_description():
    return launch.LaunchDescription([
        launch.actions.DeclareLaunchArgument('msg_A', default_value='Tick'),
        launch.actions.DeclareLaunchArgument('msg_B', default_value='Tack'),
        launch.actions.DeclareLaunchArgument('timer_period'),
        launch.actions.LogInfo(
            msg=launch.substitutions.LaunchConfiguration('msg_A')),
        launch.actions.LogInfo(
            msg=launch.substitutions.LaunchConfiguration('msg_B')),
        launch.actions.LogInfo(
            msg=launch.substitutions.LaunchConfiguration('timer_period')),
        # All the arguments have to be strings. Floats will give an error of NonIterable.
        Node(
            package='launch_args_example_pkg',
            executable='arguments_examples_demo',
            output='screen',
            emulate_tty=True,
            arguments=[
                "-timer_period_message", 
                launch.substitutions.LaunchConfiguration('msg_A'),
                launch.substitutions.LaunchConfiguration('msg_B'),
                "-timer_period", 
                launch.substitutions.LaunchConfiguration('timer_period')
            ]
        ),
    ])
    ```

5. Not sure about this syntax and its effect, specifically the role of `-timer_period_message` and `-timer_period`:
```
                arguments=[
                "-timer_period_message", 
                launch.substitutions.LaunchConfiguration('msg_A'),
                launch.substitutions.LaunchConfiguration('msg_B'),
                "-timer_period", 
                launch.substitutions.LaunchConfiguration('timer_period')
            ]
```  
Probably has to do with substitutions.  

6. One can provide all declared arguments, not just the reqiured ones:
```
user:~/ros2_ws$ ros2 launch launch_args_example_pkg start_with_arguments.launch.py msg_A:="How now..." msg_B:="Brown cow!" timer_period:=0.7
[INFO] [launch]: All log files can be found below /home/user/.ros/log/2024-08-05-05-37-41-867593-1_xterm-1892
[INFO] [launch]: Default logging verbosity is set to INFO
[INFO] [launch.user]: How now...
[INFO] [launch.user]: Brown cow!
[INFO] [launch.user]: 0.7
[INFO] [arguments_examples_demo-1]: process started with pid [1894]
[arguments_examples_demo-1] [INFO] [1722836262.709246216] [dummy_arguments_example]: --- How now... ---
[arguments_examples_demo-1] [INFO] [1722836263.409235554] [dummy_arguments_example]: --- Brown cow! ---
[arguments_examples_demo-1] [INFO] [1722836264.109235038] [dummy_arguments_example]: --- How now... ---
[arguments_examples_demo-1] [INFO] [1722836264.809238115] [dummy_arguments_example]: --- Brown cow! ---
```
