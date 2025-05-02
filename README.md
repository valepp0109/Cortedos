# Implementación de TurtleBot3 con LIDAR y SLAM en Docker

## Resumen
Este proyecto implementa un robot TurtleBot3 utilizando técnicas de SLAM (Simultaneous Localization and Mapping) con sensor LIDAR en un entorno Docker. La implementación permite la simulación, mapeo y teleoperación del robot en un entorno virtual.

## Índice
1. [Configuración del entorno](#configuración-del-entorno)
2. [Ejecución del sistema](#ejecución-del-sistema)
3. [Visualización del mapa](#visualización-del-mapa)
4. [Desafíos y consideraciones](#desafíos-y-consideraciones)
5. [Conclusiones](#conclusiones)

## Configuración del entorno
Para iniciar el proyecto, se creó un Dockerfile con las siguientes configuraciones:

dockerfile
FROM osrf/ros:noetic-desktop-full

# Instalar TurtleBot3 y SLAM
RUN apt-get update && apt-get install -y \
    ros-noetic-turtlebot3 \
    ros-noetic-turtlebot3-simulations \
    ros-noetic-slam-gmapping

# Configurar entorno
RUN echo "export TURTLEBOT3_MODEL=burger" >> ~/.bashrc

# Script de inicio
CMD ["bash", "-c", "source /opt/ros/noetic/setup.bash && \
              roslaunch turtlebot3_gazebo turtlebot3_world.launch"]


Este Dockerfile instala los paquetes necesarios para TurtleBot3, configura el modelo del robot como "burger" y establece un comando de inicio para lanzar el entorno de simulación.

## Ejecución del sistema
Para ejecutar el sistema completo, se utilizan tres terminales diferentes:

### Terminal 1: Simulación (Gazebo)
bash
docker run -it --rm \
    --env="DISPLAY" \
    -v /tmp/.X11-unix:/tmp/.X11-unix \
    --network=host \
    --name slam-bot \
    turtlebot3-slam

Este comando inicia el contenedor Docker con el entorno de simulación Gazebo, configurando la visualización gráfica y el acceso a la red.

### Terminal 2: SLAM (dentro del mismo contenedor)
bash
docker exec -it slam-bot bash -c "source /opt/ros/noetic/setup.bash && \
                     roslaunch turtlebot3_slam turtlebot3_slam.launch slam_methods:=gmapping"

Este comando ejecuta el algoritmo SLAM dentro del contenedor ya iniciado, utilizando el método gmapping para la construcción del mapa.

### Terminal 3: Teleoperación (para mover el robot)
bash
docker exec -it slam-bot bash -c "source /opt/ros/noetic/setup.bash && \
                     rosrun turtlebot3_teleop turtlebot3_teleop_key"

Este comando permite controlar el robot mediante el teclado, facilitando la exploración del entorno para el mapeo.

## Visualización del mapa
Para visualizar el mapa generado por el robot, se utiliza la herramienta rviz:

bash
docker exec -it slam-bot bash -c "source /opt/ros/noetic/setup.bash && \
                     rosrun rviz rviz -d /opt/ros/noetic/share/turtlebot3_slam/rviz/turtlebot3_gmapping.rviz"

Esta herramienta muestra de manera gráfica el mapa que se está construyendo a medida que el robot explora el entorno.

## Desafíos y consideraciones
Durante la implementación se enfrentaron varios desafíos:

- *Limitaciones de hardware*: El proceso de simulación y mapeo requiere recursos computacionales significativos. En equipos con recursos limitados, pueden producirse cierres inesperados de la aplicación.
- *Estabilidad del sistema*: La integración de Docker con aplicaciones gráficas como Gazebo y rviz puede presentar problemas de estabilidad dependiendo del sistema operativo anfitrión.
- *Rendimiento del LIDAR*: El sensor LIDAR simulado funciona correctamente, pero su rendimiento puede verse afectado por las limitaciones de hardware del equipo.

## Conclusiones
La implementación de TurtleBot3 con SLAM y LIDAR en Docker proporciona un entorno flexible y reproducible para experimentar con tecnologías de robótica. Aunque existen desafíos relacionados con los recursos computacionales necesarios, el enfoque basado en contenedores facilita la configuración y ejecución del sistema en diferentes entornos.

Para trabajos futuros, se podría considerar:
- Optimizar la configuración de Docker para reducir la carga computacional
- Explorar diferentes algoritmos de SLAM más allá de gmapping
- Implementar rutas predefinidas para la exploración autónoma del entorno
