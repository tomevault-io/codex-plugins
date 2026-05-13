<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# ROS1 Noetic RANSAC Ground Segmentation
- Node subscribes to a point cloud topic (sensor_msgs/PointCloud2)
- Uses tf to transform point cloud to base_link
- RANSAC fits ground plane
- Publishes ground and obstacle point clouds
- Use PCL and tf libraries
- Catkin-compatible

---
> Source: [lft123454321/ransac_ground_segmentation](https://github.com/lft123454321/ransac_ground_segmentation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-13 -->
