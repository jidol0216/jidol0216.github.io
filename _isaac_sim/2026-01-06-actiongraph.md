---
title: "Python으로 ActionGraph 생성"
collection: isaac_sim
date: 2026-01-06
permalink: /isaac-sim/actiongraph/
excerpt: "Python으로 Isaac Sim에서 ActionGraph를 생성하고 노드를 구성하는 예제와 노드 타입 찾기 방법."
---

## Python으로 ActionGraph 생성

### ROS2 로봇 제어 Graph 예시

```python
import omni.graph.core as og

keys = og.Controller.Keys
(graph, nodes, _, _) = og.Controller.edit(
    {"graph_path": "/World/Robot/ActionGraph", "evaluator_name": "execution"},
    {
        keys.CREATE_NODES: [
            ("OnPlaybackTick", "omni.graph.action.OnPlaybackTick"),
            ("SubscribeTwist", "omni.isaac.ros2_bridge.ROS2SubscribeTwist"),
            ("DifferentialController", "omni.isaac.wheeled_robots.DifferentialController"),
            ("ArticulationController", "omni.isaac.core_nodes.IsaacArticulationController"),
        ],
        keys.CONNECT: [
            ("OnPlaybackTick.outputs:tick", "SubscribeTwist.inputs:execIn"),
            ("SubscribeTwist.outputs:execOut", "DifferentialController.inputs:execIn"),
            ("SubscribeTwist.outputs:linearVelocity", "DifferentialController.inputs:linearVelocity"),
            ("SubscribeTwist.outputs:angularVelocity", "DifferentialController.inputs:angularVelocity"),
            ("DifferentialController.outputs:velocityCommand", "ArticulationController.inputs:velocityCommand"),
        ],
        keys.SET_VALUES: [
            ("SubscribeTwist.inputs:topicName", "/cmd_vel"),
            ("DifferentialController.inputs:wheelRadius", 0.1),
            ("DifferentialController.inputs:wheelBase", 0.5),
            ("ArticulationController.inputs:robotPath", "/World/Robot"),
        ],
    }
)
```

### 노드 타입 찾기

```python
import omni.graph.core as og
node_types = og.get_registered_node_types()
for nt in node_types:
    if "ros2" in nt.lower():
        print(nt)
```

**결론**: Python으로 Graph 생성 가능하지만, USD에 이미 Graph가 있다면 Reference로 불러오는 게 훨씬 편함!
