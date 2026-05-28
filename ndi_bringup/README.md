# Writing bringup package for your own project
If you want to use the driver in your own project, you are not advised to modify `ndi_description` and `ndi_hardware` packages as the driver is designed to put all customization and configuration in the **bringup** package. It is recommended to take `ndi_bringup` as an example and write your own bringup package. To do so, you need the following steps. 

## 1. Write your tracker configuration file
Write one tracker configuration file with reference to [config/ndi_trackers.yaml](config/ndi_trackers.yaml). The launch file reads this YAML and uses it to:
- pass tracker names and SROM paths into the Polaris URDF,
- configure `ros2_control`,
- spawn one pose broadcaster for each tracker.

The file should follow this structure:

```yaml
controller_manager:
  ros__parameters:
    update_rate: 100
    trackers:
      - name: "ee_tracker"
        srom: "$(find ndi_description)/sroms/8700339.rom"
      - name: "dental_probe"
        srom: "$(find ndi_description)/sroms/8700340.rom"
```

Each tracker `name` must be unique. The `srom` value should point to the SROM file for that tracker. The pose broadcaster for a tracker is spawned as `<tracker_name>_pose_broadcaster`.

## 2. Write your launch file
Write your launch file with reference to [launch/polaris.launch.py](launch/polaris.launch.py). Following nodes will be launched to bring up the driver:
- `robot_state_publisher/robot_state_publisher` for publishing the generated robot description.
- `controller_manager/ros2_control_node` for launching the `ros2_control` stack with your tracker YAML file.
- `controller_manager/spawner` for spawning each pose broadcaster generated from the tracker list.
- `rviz2/rviz2` to visualize broadcasted tracker poses, if needed.

The example launch file accepts a `config` argument, so you can use a custom tracker YAML without editing the launch file:

```bash
ros2 launch ndi_bringup polaris.launch.py \
  ip:=<your_ndi_ip> \
  config:=/absolute/path/to/your_ndi_trackers.yaml
```

**Reminder**: If you want to use the NDI device with other devices controlled by `ros2_control`, please be careful to configure the **namespace** of each node and topic.
