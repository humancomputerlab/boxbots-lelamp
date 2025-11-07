# Lelamp Studio

https://github.com/humancomputerlab/lelamp-studio


A web-based visual interface for animating and controlling the Lelamp robot. This React application provides a node-based graph editor to create robot animations, visualize URDF models in 3D, and export motion sequences for the robot lab.

## What It Is

The Lelamp Interface is a browser-based tool for:
- **3D Robot Visualization**: Load and view URDF robot models with interactive joint manipulation
- **Node-Based Animation Editor**: Create animation sequences using a visual graph of joint poses and transitions
- **Animation Playback**: Preview robot movements in real-time before deploying to hardware
- **CSV Export**: Generate keyframe data files compatible with robot runtime systems
- **AI-Powered Poses**: Optional integration with Neocortex AI to generate intelligent robot behaviors

## How to Use

### Setup

```bash
npm i
npm run dev
```

The application will start on `http://localhost:5173` (or your configured port).

### Basic Workflow

1. **Upload Robot Model**
   - Click "Upload Simulation" in the sidebar
   - Upload a ZIP file containing:
     - `robot.URDF` (or similar URDF file)
     - All associated mesh files (STL, DAE, etc.)
   - The robot will appear in the 3D viewer with all joints available

2. **Create Animation Sequence**
   - Use the toolbar buttons in the node graph canvas:
     - **Joint**: Create a pose node with specific joint angles
     - **Transition**: Add smooth motion between poses
     - **AI Agent**: Generate AI-powered poses (optional)
   - Click nodes to edit joint values using sliders
   - Connect nodes by dragging from output to input ports

3. **Animate and Export**
   - Click "Run Animation" to preview the sequence in the 3D viewer
   - Use "Record" to capture joint positions during playback
   - Click "Export" to download a CSV file with timestamped joint positions
   - The CSV format: `timestamp,joint1,joint2,joint3,joint4,joint5`

4. **Manual Control**
   - Select joints from the dropdown in the 3D viewer
   - Drag sliders or use the 3D manipulator to adjust joint angles
   - Changes update in real-time on the robot model

## Connecting to Robot Lab

### CSV Export Format

The exported CSV files contain keyframe data that can be consumed by the robot runtime:

```csv
timestamp,joint1,joint2,joint3,joint4,joint5
0,0.0,0.0,0.0,0.0,0.0
20,0.5,0.3,0.2,0.1,0.0
40,1.0,0.6,0.4,0.2,0.0
...
```

- **Timestamp**: Milliseconds from animation start
- **Joint values**: Radian angles for each joint (1-5 for Lelamp)

### Integration Points

1. **CSV Import/Export**: The interface can import existing CSV animations and export new ones. Use the exported CSV files with your robot runtime system.

2. **Joint State Management**: Joint values are managed via Zustand store (`src/store/useJointStore.ts`). This can be extended to send real-time commands to robot hardware via WebSocket or HTTP API.

3. **Runtime Connection** (Future): To connect directly to the robot lab:
   - Add a WebSocket client in `src/lib/robotRuntime.ts` (create this file)
   - Subscribe to joint value changes from the store
   - Send joint commands to the robot runtime endpoint
   - Implement the RGB service interface mentioned in the codebase

### Example Integration Pattern

```typescript
// Example: Send joint values to robot runtime
import { useJointStore } from '@/store/useJointStore';

const sendToRobot = (jointValues: Record<string, number>) => {
  fetch('http://robot-lab-ip:port/api/joints', {
    method: 'POST',
    body: JSON.stringify(jointValues)
  });
};

// Subscribe to joint changes
useJointStore.subscribe(
  (state) => state.jointValues,
  (jointValues) => sendToRobot(jointValues)
);
```

## Features

- ✅ URDF loader with mesh support
- ✅ Interactive 3D joint manipulation
- ✅ Node-based animation graph editor
- ✅ Smooth transition interpolation
- ✅ Animation recording and CSV export
- ✅ CSV import for existing animations
- ✅ AI-powered pose generation (Neocortex integration)
- ✅ Real-time 3D preview

## Tech Stack

- **React 18** + **TypeScript**
- **React Three Fiber** (3D rendering)
- **React Flow** (node graph editor)
- **URDF Loader** (robot model parsing)
- **Zustand** (state management)
- **Vite** (build tool)

## Project Structure

- `src/components/Viewer3D.tsx` - 3D robot visualization
- `src/components/NodeGraph.tsx` - Animation graph editor
- `src/store/useJointStore.ts` - Joint state management
- `src/lib/neocortex.ts` - AI integration (optional)

## Notes

- The interface is designed for 5-DOF robots (Lelamp), but can work with any URDF model
- Joint orientation mapping may need calibration for your specific robot
- RGB service integration is planned but not yet implemented
- See `NEOCORTEX_INTEGRATION.md` for AI features documentation