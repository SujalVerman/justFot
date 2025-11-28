# How to Use StackedBarWithLine Chart

## Step 1: Import the Component

```typescript
import StackedBarWithLine1 from '@/components/mf/charts/StackedBarwithLine1';
```

---

## Step 2: Create a Hook for API Call

Create a hooks file (e.g., `hooks/ChartApi.ts`):

```typescript
import { useApi } from "../../api-base";  // Adjust path based on your folder structure
import Endpoint from "@/common/endpoint";

export const useGetChartData = (
  payload: {
    package_name: string;
    start_date?: Date | string;
    end_date?: Date | string;
    frequency?: string;
  },
  enabled: boolean = true
) => {
  return useApi<any[]>(
    `${process.env.NEXT_PUBLIC_YOUR_API_BASE}/your-endpoint`,
    "POST",
    payload,
    {
      queryKey: [
        "getChartData", 
        payload.package_name, 
        payload.start_date?.toString() || "", 
        payload.end_date?.toString() || ""
      ],
      enabled,
      refetchOnMount: true,
    }
  );
};
```

---

## Step 3: Use the Hook in Your Component

```typescript
"use client";
import React, { useState, useEffect } from "react";
import StackedBarWithLine1 from '@/components/mf/charts/StackedBarwithLine1';
import { useGetChartData } from "./hooks/ChartApi";
import { usePackage } from "@/components/mf/PackageContext";
import { useDateRange } from "@/components/mf/DateRangeContext";

const MyPage = () => {
  // Get context values
  const { selectedPackage } = usePackage();
  const { startDate, endDate } = useDateRange();
  
  // State for chart data
  const [chartData, setChartData] = useState<any[]>([]);
  
  // Prepare API payload
  const chartPayload = {
    package_name: selectedPackage || "",
    start_date: startDate,
    end_date: endDate,
    frequency: "daily",
  };
  
  // Check if we have valid data
  const hasValidPackage = !!selectedPackage;
  
  // Call the hook
  const { data: chartApiData, isLoading, refetch } = useGetChartData(
    chartPayload,
    hasValidPackage
  );
  
  // Transform API response to chart data format
  useEffect(() => {
    if (chartApiData && Array.isArray(chartApiData)) {
      const mappedData = chartApiData.map((item: any) => ({
        month: item.month || item.date || item.week,  // X-axis value
        "Total Count": item.total_count || 0,        // Bar value 1
        "Fraud Count": item.fraud_count || 0,        // Bar value 2 (stacks on top)
        "Fraud %": typeof item.fraud_percentage === "string" 
          ? parseFloat(item.fraud_percentage.replace("%", "")) 
          : item.fraud_percentage || 0,               // Line value (must be number)
      }));
      setChartData(mappedData);
    } else {
      setChartData([]);
    }
  }, [chartApiData]);
  
  // Chart configuration
  const chartConfig = {
    "Total Count": { 
      label: "Valid Count", 
      color: "#00A86B" 
    },
    "Fraud Count": { 
      label: "Invalid Count", 
      color: "#FF0000" 
    },
    "Fraud %": { 
      label: "Invalid %", 
      color: "#b91c1c" 
    },
  };

  return (
    <StackedBarWithLine1
      chartData={chartData}
      chartConfig={chartConfig}
      isLoading={isLoading}
      isLegend={true}
      isHorizontal={false}
      xAxisConfig={{
        dataKey: "month",
        tickLine: false,
        tickMargin: 10,
        axisLine: false,
        tickFormatter: (value: string) => value,
        textAnchor: "middle",
        dy: 10,
      }}
      YAxis1={{
        yAxisId: "left",
        orientation: "left",
        stroke: "hsl(var(--chart-1))",
      }}
      YAxis2={{
        yAxisId: "right",
        orientation: "right",
        stroke: "hsl(var(--chart-3))",
      }}
      onExpand={() => {}}
    />
  );
};

export default MyPage;
```

---

## Component Props Explained

### Required Props

| Prop | Type | Description | Example |
|------|------|-------------|---------|
| `chartData` | `any[]` | Array of data objects for the chart | `[{month: "Jan", "Total Count": 100, ...}]` |
| `chartConfig` | `Object` | Configuration mapping data keys to labels and colors | `{"Total Count": {label: "...", color: "..."}}` |
| `onExpand` | `() => void` | Callback function (can be empty) | `() => {}` |

### Important Optional Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `isLoading` | `boolean` | `false` | Shows loading spinner when `true` |
| `isLegend` | `boolean` | `true` | Shows/hides the legend |
| `isHorizontal` | `boolean` | `false` | Chart orientation (`false` = vertical) |
| `xAxisConfig` | `Object` | - | X-axis configuration (see below) |
| `YAxis1` | `Object` | - | Left Y-axis configuration (for bars) |
| `YAxis2` | `Object` | - | Right Y-axis configuration (for line) |

### X-Axis Configuration

```typescript
xAxisConfig={{
  dataKey: "month",              // Key from your data object for X-axis
  tickLine: false,                // Show/hide tick lines
  tickMargin: 10,                 // Margin for ticks
  axisLine: false,                // Show/hide axis line
  tickFormatter: (value: string) => value,  // Format tick labels
  textAnchor: "middle",           // Text alignment
  dy: 10,                         // Vertical offset
}}
```

### Y-Axis Configuration

**Left Y-Axis (for stacked bars):**
```typescript
YAxis1={{
  yAxisId: "left",
  orientation: "left",
  stroke: "hsl(var(--chart-1))",
}}
```

**Right Y-Axis (for line chart):**
```typescript
YAxis2={{
  yAxisId: "right",
  orientation: "right",
  stroke: "hsl(var(--chart-3))",
}}
```

---

## Data Format

### Chart Data Structure

Each object in the `chartData` array represents one point on the chart:

```typescript
[
  {
    month: "January",           // X-axis value (can be any key name)
    "Total Count": 100,         // Bar value 1 (left Y-axis)
    "Fraud Count": 20,          // Bar value 2 (stacks on top, left Y-axis)
    "Fraud %": 20.0             // Line value (right Y-axis, must be number)
  },
  {
    month: "February",
    "Total Count": 150,
    "Fraud Count": 30,
    "Fraud %": 20.0
  }
]
```

### Chart Config Structure

The `chartConfig` object maps data keys to display labels and colors:

```typescript
{
  "Total Count": { 
    label: "Valid Count",      // What shows in legend
    color: "#00A86B"           // Color for this bar
  },
  "Fraud Count": { 
    label: "Invalid Count", 
    color: "#FF0000" 
  },
  "Fraud %": { 
    label: "Invalid %", 
    color: "#b91c1c"          // Color for the line
  },
}
```

**Important Rules:**
- Keys in `chartConfig` must match keys in your `chartData` objects
- The percentage key (e.g., `"Fraud %"`) will be used for the **line chart** (right Y-axis)
- Other keys will be used for **stacked bars** (left Y-axis)
- Percentage values must be **numbers**, not strings (e.g., `20.0` not `"20%"`)

---

## API Data Transformation

### Sending Data to API

The payload you send to your API hook:

```typescript
const chartPayload = {
  package_name: selectedPackage || "",
  start_date: startDate,
  end_date: endDate,
  frequency: "daily",  // or "weekly", "monthly"
};
```

### Receiving and Transforming API Response

Transform the API response to match the chart data format:

```typescript
useEffect(() => {
  if (chartApiData && Array.isArray(chartApiData)) {
    const mappedData = chartApiData.map((item: any) => ({
      // X-axis value (adjust based on your API response)
      month: item.month || item.date || item.week,
      
      // Bar values (adjust based on your API response)
      "Total Count": item.total_count || 0,
      "Fraud Count": item.fraud_count || 0,
      
      // Line value - must convert string percentages to numbers
      "Fraud %": typeof item.fraud_percentage === "string"
        ? parseFloat(item.fraud_percentage.replace("%", ""))
        : item.fraud_percentage || 0,
    }));
    
    setChartData(mappedData);
  } else {
    setChartData([]);
  }
}, [chartApiData]);
```

### Common API Response Formats

**Format 1: Separate fields**
```typescript
// API returns:
{ month: "2024-01", total_count: 100, fraud_count: 20, fraud_percentage: "20%" }

// Transform to:
{ month: "2024-01", "Total Count": 100, "Fraud Count": 20, "Fraud %": 20.0 }
```

**Format 2: Combined count**
```typescript
// API returns:
{ date: "2024-01-01", clean_count: 80, fraud_count: 20, fraud_percentage: 20.0 }

// Transform to:
{ 
  month: "2024-01-01", 
  "Total Count": 100,  // clean_count + fraud_count
  "Fraud Count": 20, 
  "Fraud %": 20.0 
}
```

---

## Complete Working Example

```typescript
"use client";
import React, { useState, useEffect } from "react";
import StackedBarWithLine1 from '@/components/mf/charts/StackedBarwithLine1';
import { useGetChartData } from "./hooks/ChartApi";
import { usePackage } from "@/components/mf/PackageContext";
import { useDateRange } from "@/components/mf/DateRangeContext";

const MyPage = () => {
  const { selectedPackage } = usePackage();
  const { startDate, endDate } = useDateRange();
  const [chartData, setChartData] = useState<any[]>([]);

  const chartPayload = {
    package_name: selectedPackage || "",
    start_date: startDate,
    end_date: endDate,
  };

  const { data: chartApiData, isLoading } = useGetChartData(
    chartPayload,
    !!selectedPackage
  );

  useEffect(() => {
    if (chartApiData && Array.isArray(chartApiData)) {
      const mappedData = chartApiData.map((item: any) => ({
        month: item.month,
        "Total Count": item.total_count || 0,
        "Fraud Count": item.fraud_count || 0,
        "Fraud %": parseFloat((item.fraud_percentage || "0").replace("%", "")),
      }));
      setChartData(mappedData);
    }
  }, [chartApiData]);

  const chartConfig = {
    "Total Count": { label: "Valid Count", color: "#00A86B" },
    "Fraud Count": { label: "Invalid Count", color: "#FF0000" },
    "Fraud %": { label: "Invalid %", color: "#b91c1c" },
  };

  return (
    <StackedBarWithLine1
      chartData={chartData}
      chartConfig={chartConfig}
      isLoading={isLoading}
      isLegend={true}
      xAxisConfig={{ dataKey: "month" }}
      YAxis1={{ yAxisId: "left", orientation: "left", stroke: "hsl(var(--chart-1))" }}
      YAxis2={{ yAxisId: "right", orientation: "right", stroke: "hsl(var(--chart-3))" }}
      onExpand={() => {}}
    />
  );
};

export default MyPage;
```

---

## Quick Checklist

- [ ] Import `StackedBarWithLine1` component
- [ ] Create hook file with `useApi` for API call
- [ ] Create `chartData` state
- [ ] Create `chartConfig` object
- [ ] Call hook with payload and enabled condition
- [ ] Transform API response in `useEffect`
- [ ] Match keys between `chartData` and `chartConfig`
- [ ] Convert percentages to numbers (not strings)
- [ ] Set `xAxisConfig.dataKey` to match your data key
- [ ] Include both `YAxis1` and `YAxis2` props
