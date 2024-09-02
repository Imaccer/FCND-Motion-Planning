## Project: 3D Motion Planning
![Quad Image](./misc/enroute.png)

---


# Required Steps for a Passing Submission:
1. Load the 2.5D map in the colliders.csv file describing the environment.
2. Discretize the environment into a grid or graph representation.
3. Define the start and goal locations.
4. Perform a search using A* or other search algorithm.
5. Use a collinearity test or ray tracing method (like Bresenham) to remove unnecessary waypoints.
6. Return waypoints in local ECEF coordinates (format for `self.all_waypoints` is [N, E, altitude, heading], where the drone’s start location corresponds to [0, 0, 0, 0].
7. Write it up.
8. Congratulations!  Your Done!

## [Rubric](https://review.udacity.com/#!/rubrics/1534/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  

You're reading it! Below I describe how I addressed each rubric point and where in my code each point is handled.

### Explain the Starter Code

#### 1. Explain the functionality of what's provided in `motion_planning.py` and `planning_utils.py`
 The function, `plan_path`, is responsible for planning a path for the drone to follow, primarily using a grid-based approach. Here's an explanation of the function:

The function starts by setting the drone's flight state to PLANNING

The target altitude is then set to 5 meters, and the safety distance is set to 5 meters to keep the drone away from obstacles.

The target position's altitude component is set to the defined TARGET_ALTITUDE.

The function reads the obstacle map data from colliders.csv, which contains information about obstacles in the environment. It skips the first two rows and reads the rest of the data into a NumPy array.

The function calls the `create_grid` function (defined in `planning_utils.py`) with the obstacle data, target altitude, and safety distance to create a 2D grid representation of the environment. Obstacles are marked, and free space is defined for the drone to navigate.
It returns the grid, north offset, and east offset values, which help align the grid with the real-world coordinate system.
Define Start Position on the Grid:

The starting position on the grid is set to the center, defined by the negative offsets of the grid.

The goal position on the grid is set arbitrarily, offset by 10 units north and east from the grid's start position.

The A* algorithm is implemented as the `a_star` function in `planning_utils.py` is used to find the optimal path from the start position to the goal position on the grid. It takes the grid, a heuristic, and the start and goal coordinates that we want to plan a path between. The path is returned as a sequence of coordinates.

The path coordinates are converted into waypoints, adjusting for the north and east offsets and setting the target altitude for each waypoint.

The generated waypoints are stored in self.waypoints.

The `send_waypoints()` method is called to send these waypoints to the simulator, enabling visualization of the planned path.
#### Summary
Overall, this function sets up a planning environment, creates a grid map based on obstacles, determines a path using A* search, and prepares waypoints for the drone to follow.
<!-- And here's a lovely image of my results (ok this image has nothing to do with it, but it's a nice example of how to include images in your writeup!)
![Top Down View](./misc/high_up.png)

Here's | A | Snappy | Table
--- | --- | --- | ---
1 | `highlight` | **bold** | 7.41
2 | a | b | c
3 | *italic* | text | 403
4 | 2 | 3 | abcd -->

### Implementing Your Path Planning Algorithm

#### 1. Set your global home position
Here students should read the first line of the csv file, extract lat0 and lon0 as floating point values and use the self.set_home_position() method to set global home. Explain briefly how you accomplished this in your code.

This code reads the first line of a file named colliders.csv and extracts initial latitude and longitude values. It opens the file in read mode using a with statement to ensure proper closure after reading. The readline() method reads the first line, which is expected to contain position data, and split(",") separates the line into a list based on commas. To extract the latitude (lat0), the code accesses the first item of the list, removes any extra whitespace, splits the string by spaces, retrieves the numeric part, and converts it to a floating-point number. The process is repeated for the longitude (lon0) by applying the same steps to the second item of the list. 

The latitude and longitude, along with 0 for the altitude, are passed to the `set_home_position()` method of the parent `Drone` class. 

<!-- And here is a lovely picture of our downtown San Francisco environment from above!
![Map of SF](./misc/map.png) -->

#### 2. Set your current local position
Here as long as you successfully determine your local position relative to global home you'll be all set. Explain briefly how you accomplished this in your code.

Firstly,  the global home position was obtained from the parent `Drone` class properties _latitude, _longitude, _altitude. 

Then the function defined in the udacidrone library (from `udacidrone.frame_utils` import `global_to_local`
) for conversion of global to local coordinates is used to convert to current local position. 
<!-- 
Meanwhile, here's a picture of me flying through the trees!
![Forest Flying](./misc/in_the_trees.png) -->

#### 3. Set grid start position from local position
This is another step in adding flexibility to the start location. As long as it works you're good to go!

#### 4. Set grid goal position from geodetic coords
This step is to add flexibility to the desired goal location. Should be able to choose any (lat, lon) within the map and have it rendered to a goal location on the grid.

#### 5. Modify A* to include diagonal motion (or replace A* altogether)
Minimal requirement here is to modify the code in planning_utils() to update the A* implementation to include diagonal motions on the grid that have a cost of sqrt(2), but more creative solutions are welcome. Explain the code you used to accomplish this step.

Added diagonal steps to the `Action` class, for example:    SOUTH_EAST = (1, 1, np.sqrt(2)), where the last element is the cost of the diagonal step.

Also updated `valid_actions` to add checks for the diagonal motions to see if they're valid for the grid and current cell being evaluated.

#### 6. Cull waypoints 
For this step you can use a collinearity test or ray tracing method like Bresenham. The idea is simply to prune your path of unnecessary waypoints. Explain the code you used to accomplish this step.

The `prune_path` function simplifies a path by removing unnecessary intermediate points that lie on a straight line between their neighbors, reducing the total number of waypoints without altering the overall path shape. It starts with the first point and iterates through the path, examining sets of three consecutive points at a time. If the three points are collinear (i.e., they lie on a straight line), the middle point is excluded; otherwise, it is retained because it indicates a direction change. The final point of the path is always included to ensure the path's endpoint remains accurate. This approach results in a pruned path with only the critical waypoints needed to preserve the path's essential direction and turns.

To check the collinearity of any set of 3 consecutive points along the path, the following method was used:

        Check if the area of the triangle formed by the three points is close to zero
        This can be done using the determinant of the matrix formed by these points
        | x1 y1 1 |
        | x2 y2 1 |
        | x3 y3 1 |
        If determinant is zero, points are collinear

Note, assumed constant altitude.

### Execute the flight
#### 1. Does it work?
It works!

### Double check that you've met specifications for each of the [rubric](https://review.udacity.com/#!/rubrics/1534/view) points.
  
# Extra Challenges: Real World Planning

For an extra challenge, consider implementing some of the techniques described in the "Real World Planning" lesson. You could try implementing a vehicle model to take dynamic constraints into account, or implement a replanning method to invoke if you get off course or encounter unexpected obstacles.


