import React, { useState } from "react";
import { Button } from "@/components/ui/button";
import { Card, CardContent } from "@/components/ui/card";
import { useEffect } from "react";
import io from "socket.io-client";
import nipplejs from "nipplejs";

const socket = io("http://localhost:5000");

export default function DroneControl() {
  const [uvLight, setUvLight] = useState(false);

  useEffect(() => {
    const joystickZone = document.getElementById("joystick-zone");
    const manager = nipplejs.create({
      zone: joystickZone,
      mode: "dynamic",
      color: "blue",
    });

    manager.on("move", (evt, data) => {
      socket.emit("move", { angle: data.angle.degree, force: data.force });
    });

    manager.on("end", () => {
      socket.emit("move", { angle: 0, force: 0 });
    });
  }, []);

  const toggleUVLight = () => {
    setUvLight(!uvLight);
    socket.emit("toggle_uv", { state: !uvLight });
  };

  return (
    <div className="p-6 grid gap-6">
      <h1 className="text-xl font-bold">UV Sanitization Drone Control</h1>
      <Card>
        <CardContent className="flex flex-col items-center gap-4">
          <div id="joystick-zone" className="h-40 w-40 bg-gray-200 rounded-full"></div>
          <Button onClick={toggleUVLight} className={uvLight ? "bg-green-500" : "bg-red-500"}>
            {uvLight ? "Turn UV Off" : "Turn UV On"}
          </Button>
        </CardContent>
      </Card>
    </div>
  );
}
