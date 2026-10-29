# Laboratorio de Robótica: UR5 con gripper en ROS 2 y MoveIt 2

Escuela Colombiana de Ingeniería Julio Garavito, Robótica (RBTC).
Samuel Jerónimo Moreno Mora y Joshua Tapias. Profesor: Daniel Palomino.

Paquetes base: [ur5_gripper_packages](https://github.com/DanielPalominoS/ur5_gripper_packages).

## Contenido

| Carpeta | Qué contiene |
|---|---|
| [`informe/`](informe/) | Informe en formato IEEE (`informe_ur5_ieee.pdf`) y su fuente LaTeX |
| [`ur5_moveit_trajectory/`](ur5_moveit_trajectory/) | **Actividad 4.** Nodo C++ con `MoveGroupInterface` y `PlanningSceneInterface` que ejecuta la rutina Pick and Place |
| [`ur5_robotiq_moveit/`](ur5_robotiq_moveit/) | Configuración de MoveIt usada en la actividad 4 (estructura de MoveIt Setup Assistant con las correcciones del informe) |
| [`videos/actividad_4.mp4`](videos/actividad_4.mp4) | Video de la actividad 4: corrida normal y corrida con un pilar en el camino |
| [`resultados/`](resultados/) | Registros, rosbags, análisis y corridas fallidas documentadas |

Los videos de las actividades 1 a 3 y el nodo de Pick and Place por tópicos están en
[ur5_pick_and_place](https://github.com/Samuel-morenom/ur5_pick_and_place).

## Resultados de la actividad 4

- Escena normal: **4 de 4 corridas** completaron las 9 etapas; 36 trayectorias ejecutadas,
  planes calculados en 14 a 61 ms, trayectorias cartesianas resueltas al 100 %.
- Con un pilar de 0,40 m entre P2 y P3, MoveIt lo rodea: la aproximación a P3 pasa de
  13 puntos / 1,12 s a 24 puntos / 2,21 s y el retorno a Home de 1,63 s a 5,44 s.
- Factor de tiempo real de Gazebo con `move_group` y RViz: 0,40 a 0,48.

Entorno de ejecución: Ubuntu 24.04, ROS 2 Kilted (RoboStack), MoveIt 2.14.3,
Gazebo Ionic 9.5, `gz_ros2_control` 2.0.17, 2 núcleos, sin GPU.

## Cómo ejecutarlo

```bash
cd ~/ur5_ws/src
git clone https://github.com/DanielPalominoS/ur5_gripper_packages.git
git clone https://github.com/tapiasjoshua/lab_robotica-.git
cd ~/ur5_ws
rosdep install --from-paths src --ignore-src -r -y --skip-keys robotiq_description
colcon build && source install/setup.bash

# Antes de empezar: no deben quedar puentes de reloj de corridas anteriores (dificultad 10)
pkill -f parameter_bridge

# Terminal 1: simulación sola; esperar /joint_states
ros2 launch ur5_robotiq_bringup sim_gz_sgb.launch.py launch_rviz:=false
ros2 topic info /clock      # debe decir "Publisher count: 1"

# Terminal 2: move_group + RViz
ros2 launch ur5_moveit_trajectory move_group_sim.launch.py

# Terminal 3: rutina (add_obstacle:=true agrega el pilar)
ros2 launch ur5_moveit_trajectory moveit_routine.launch.py
```

## Problemas encontrados en la actividad 4

1. **Relojes duplicados.** Procesos `parameter_bridge` huérfanos de corridas anteriores
   republicaban `/clock`; el tiempo parecía retroceder y el controlador abortaba con
   `PATH_TOLERANCE_VIOLATED` o MoveIt rechazaba la trayectoria. Ver
   `resultados/corridas_fallidas/reloj_duplicado/`.
2. **OMPL atraviesa un obstáculo delgado.** Con la resolución por defecto, el pilar quedaba
   entre dos estados verificados y `ValidateSolution` rechazaba el plan. Se resolvió con
   `longest_valid_segment_fraction: 0.0005` en `ur5_robotiq_moveit/config/ompl_planning.yaml`.
   Ver `resultados/corridas_fallidas/pilar_resolucion_ompl/`.
