TP.DN: 16.03764412776707, 108.13903498082125
TP.HCM: 10.730332099999638, 106.67043235379343
TP.HN: 21.020575168113165, 105.83387351849966
netsh advfirewall firewall add rule name="Allow ICMPv4-In" protocol=icmpv4:8,any dir=in action=allow

from(bucket: "librenms")
  |> range(start: -5m)
  |> filter(fn: (r) => r._measurement == "devices")
  |> filter(fn: (r) => r._field == "status" or r._field == "uptime" or r._field == "ping")
  |> yield(name: "device_status")

from(bucket: "librenms")
  |> range(start: -10m)
  |> filter(fn: (r) => r._measurement == "devices" and r._field == "status")
  |> filter(fn: (r) => r._value == 0)
  |> yield(name: "down_devices")

from(bucket: "librenms")
  |> range(start: -5m)
  |> filter(fn: (r) => r._measurement == "ports")
  |> filter(fn: (r) => r._field == "bps_in" or r._field == "bps_out")
  |> yield(name: "network_traffic")
  
from(bucket: "librenms")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "ports" and (r._field == "bps_in" or r._field == "bps_out"))
  |> group(columns: ["ifName"])
  |> max()
  |> yield(name: "max_bandwidth")
