# How to Use StackedBarWithLine Chart - Beginner's Guide

## What is StackedBarWithLine Chart?

The `StackedBarWithLine1` component is a chart that shows:
- **Stacked bars** (multiple colored bars stacked on top of each other) - usually for counts/numbers
- **Line chart** (a line going across) - usually for percentages or prices
- **Two Y-axes** - Left side for bars, Right side for the line

Think of it like showing both "how many items" (bars) and "what percentage" (line) at the same time!

---

## Step-by-Step Guide: How to Use in Any Page

### Step 1: Import the Component

At the top of your page file, add this import:

```typescript
import StackedBarWithLine1 from '@/components/mf/charts/StackedBarwithLine1';
```

**Example:**
```typescript
"use client";
import React, { useState, useEffect } from "react";
import StackedBarWithLine1 from '@/components/mf/charts/StackedBarwithLine1';
// ... other imports
```

---

### Step 2: Prepare Your Chart Data State

Create a state variable to store your chart data. This will be an array of objects.

```typescript
const [chartData, setChartData] = useState<any[]>([]);
```

**What goes in this data?**
Each object in the array represents one point on the chart. For example:
```typescript
[
  {
    month: "January",        // This will be on X-axis (bottom)
    "Total Count": 100,      // This will be a bar
    "Fraud Count": 20,       // This will be stacked on top
    "Fraud %": 20.0          // This will be the line
  },
  {
    month: "February",
    "Total Count": 150,
    "Fraud Count": 30,
    "Fraud %": 20.0
  }
  // ... more months
]
```

---

### Step 3: Create Chart Configuration

Create a configuration object that tells the chart:
- What each data key means (label)
- What color to use for each

```typescript
const chartConfig = {
  "Total Count": { 
    label: "Valid Count",      // What shows in legend
    color: "#00A86B"           // Green color
  },
  "Fraud Count": { 
    label: "Invalid Count", 
    color: "#FF0000"           // Red color
  },
  "Fraud %": { 
    label: "Invalid %", 
    color: "#b91c1c"           // Dark red for line
  },
};
```

**Important:** 
- The keys (like `"Total Count"`) must match the keys in your `chartData` objects
- The `"Fraud %"` key will be used for the **line chart** (right Y-axis)
- Other keys will be used for **stacked bars** (left Y-axis)

---

### Step 4: Fetch Data from API (Optional)

If you need to get data from an API, use `useApiCall` or your API hook:

```typescript
const { result: apiResult, loading: isLoading } = useApiCall<any[]>({
  url: "YOUR_API_URL",
  method: "POST",
  manual: true,
  onSuccess: (response) => {
    // Transform API response to match your chart data format
    const mappedData = response.map((item: any) => ({
      month: item.month,                    // or item.date, item.week, etc.
      "Total Count": item.total_count,
      "Fraud Count": item.fraud_count,
      "Fraud %": parseFloat(item.fraud_percentage.replace("%", "")),  // Convert "20%" to 20
    }));
    setChartData(mappedData);
  },
  onError: () => {
    setChartData([]);  // Set empty array on error
  },
});

// Trigger the API call
useEffect(() => {
  if (isMutation(apiResult)) {
    apiResult.mutate({
      // Your API parameters here
      start_date: startDate,
      end_date: endDate,
      package_name: selectedPackage,
    });
  }
}, [startDate, endDate, selectedPackage]);
```

---

### Step 5: Add the Chart Component to Your JSX

Now add the chart component where you want it to appear:

```typescript
<StackedBarWithLine1
  chartData={chartData}
  chartConfig={chartConfig}
  isLoading={isLoading}
  isLegend={true}
  isHorizontal={false}
  xAxisConfig={{
    dataKey: "month",           // Which key from your data to use for X-axis
    tickLine: false,
    tickMargin: 10,
    axisLine: false,
    tickFormatter: (value: string) => value,
    textAnchor: "middle",
    dy: 10,
  }}
  YAxis1={{
    yAxisId: "left",            // Left Y-axis for stacked bars
    orientation: "left",
    stroke: "hsl(var(--chart-1))",
  }}
  YAxis2={{
    yAxisId: "right",           // Right Y-axis for line chart
    orientation: "right",
    stroke: "hsl(var(--chart-3))",
  }}
  onExpand={() => {}}
/>
```

---

## Complete Example

Here's a complete working example you can copy and modify:

```typescript
"use client";
import React, { useState, useEffect } from "react";
import StackedBarWithLine1 from '@/components/mf/charts/StackedBarwithLine1';
import { Card, CardContent, CardTitle } from "@/components/ui/card";

const MyPage = () => {
  // Step 2: Prepare chart data state
  const [chartData, setChartData] = useState<any[]>([]);
  const [isLoading, setIsLoading] = useState(false);

  // Step 3: Create chart configuration
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

  // Step 4: Fetch data (example with mock data)
  useEffect(() => {
    setIsLoading(true);
    // Simulate API call
    setTimeout(() => {
      const mockData = [
        { month: "Jan", "Total Count": 100, "Fraud Count": 20, "Fraud %": 20.0 },
        { month: "Feb", "Total Count": 150, "Fraud Count": 30, "Fraud %": 20.0 },
        { month: "Mar", "Total Count": 120, "Fraud Count": 24, "Fraud %": 20.0 },
        { month: "Apr", "Total Count": 180, "Fraud Count": 36, "Fraud %": 20.0 },
      ];
      setChartData(mockData);
      setIsLoading(false);
    }, 1000);
  }, []);

  return (
    <div className="p-4">
      <Card>
        <CardTitle>My Chart</CardTitle>
        <CardContent>
          {/* Step 5: Add the chart */}
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
        </CardContent>
      </Card>
    </div>
  );
};

export default MyPage;
```

---

## Understanding the Props

### Required Props (Must Have)

| Prop | What It Does | Example |
|------|--------------|---------|
| `chartData` | Your data array | `[{month: "Jan", "Total Count": 100, ...}]` |
| `chartConfig` | Configuration object | `{"Total Count": {label: "...", color: "..."}}` |
| `onExpand` | Function (can be empty) | `() => {}` |

### Important Optional Props

| Prop | What It Does | When to Use |
|------|--------------|-------------|
| `isLoading` | Shows loading spinner | `true` when fetching data |
| `isLegend` | Shows/hides legend | `true` to show what each color means |
| `isHorizontal` | Chart orientation | `false` for vertical (default) |
| `xAxisConfig` | X-axis settings | Always include this |
| `YAxis1` | Left Y-axis (for bars) | Always include this |
| `YAxis2` | Right Y-axis (for line) | Always include this |

---

## Data Format Rules

### ✅ Correct Data Format

```typescript
[
  {
    month: "January",           // X-axis value (can be any name)
    "Total Count": 100,         // Bar value 1
    "Fraud Count": 20,          // Bar value 2 (stacks on top)
    "Fraud %": 20.0             // Line value (must be number, not string)
  }
]
```

### ❌ Common Mistakes

1. **Wrong percentage format:**
   ```typescript
   "Fraud %": "20%"  // ❌ String - won't work
   "Fraud %": 20.0   // ✅ Number - correct
   ```

2. **Missing keys in chartConfig:**
   ```typescript
   // If your data has "Total Count", chartConfig MUST have it too
   chartConfig = {
     "Total Count": {...},  // ✅ Must match data key
   }
   ```

3. **Wrong dataKey in xAxisConfig:**
   ```typescript
   // If your data uses "month", xAxisConfig must use "month"
   xAxisConfig={{
     dataKey: "month",  // ✅ Must match your data key
   }}
   ```

---

## Quick Reference: Copy-Paste Template

```typescript
// 1. Import
import StackedBarWithLine1 from '@/components/mf/charts/StackedBarwithLine1';

// 2. State
const [chartData, setChartData] = useState<any[]>([]);

// 3. Config
const chartConfig = {
  "Bar1": { label: "Label 1", color: "#00A86B" },
  "Bar2": { label: "Label 2", color: "#FF0000" },
  "Line": { label: "Line Label", color: "#b91c1c" },
};

// 4. Component
<StackedBarWithLine1
  chartData={chartData}
  chartConfig={chartConfig}
  isLoading={false}
  isLegend={true}
  xAxisConfig={{ dataKey: "yourXAxisKey" }}
  YAxis1={{ yAxisId: "left", orientation: "left", stroke: "hsl(var(--chart-1))" }}
  YAxis2={{ yAxisId: "right", orientation: "right", stroke: "hsl(var(--chart-3))" }}
  onExpand={() => {}}
/>
```

---

## Real Example from opt.tsx

### Example 1: Date Wise Trend

**Data Structure:**
```typescript
[
  { month: "2024-01", "Total Count": 1000, "Fraud Count": 200, "Fraud %": 20.0 },
  { month: "2024-02", "Total Count": 1200, "Fraud Count": 240, "Fraud %": 20.0 },
]
```

**Config:**
```typescript
{
  "Total Count": { label: "Valid Count", color: "#00A86B" },
  "Fraud Count": { label: "Invalid Count", color: "#FF0000" },
  "Fraud %": { label: "Invalid %", color: "#b91c1c" },
}
```

**Result:** 
- Green and red stacked bars showing counts
- Dark red line showing percentage

### Example 2: Publisher/Vendor Wise

**Data Structure:**
```typescript
[
  { name: "Publisher A", "Clean Count": 500, "Fraud Count": 100, "Fraud %": 20.0 },
  { name: "Publisher B", "Clean Count": 300, "Fraud Count": 60, "Fraud %": 20.0 },
]
```

**Config:**
```typescript
{
  "Clean Count": { label: "Valid Count", color: "#00A86B" },
  "Fraud Count": { label: "Invalid Count", color: "#FF0000" },
  "Fraud %": { label: "Invalid %", color: "#b91c1c" },
}
```

**Result:**
- Stacked bars per publisher/vendor
- Line showing fraud percentage

---

## Tips for Beginners

1. **Start Simple**: Begin with mock data (hardcoded array) before connecting to API
2. **Check Your Keys**: Make sure keys in `chartData` match keys in `chartConfig`
3. **Numbers Not Strings**: Percentages must be numbers (20.0), not strings ("20%")
4. **Test with Sample Data**: Use 2-3 data points first to see if it works
5. **Check Console**: If chart doesn't show, check browser console for errors

---

## Troubleshooting

### Chart Not Showing?
- ✅ Check if `chartData` has data: `console.log(chartData)`
- ✅ Check if `chartConfig` keys match data keys
- ✅ Check if `xAxisConfig.dataKey` matches your data key

### Wrong Colors?
- ✅ Check `chartConfig` - each key needs a `color` property

### Line Not Showing?
- ✅ Make sure one key in `chartConfig` is for the line (usually percentage)
- ✅ Check if the value is a number, not a string

### Bars Not Stacking?
- ✅ Make sure you have multiple keys in `chartConfig` (except the line key)
- ✅ All bar values should be numbers

---

## Summary Checklist

Before using the chart, make sure you have:

- [ ] Imported the component
- [ ] Created `chartData` state
- [ ] Created `chartConfig` object
- [ ] Matched keys between data and config
- [ ] Set `xAxisConfig.dataKey` correctly
- [ ] Added both `YAxis1` and `YAxis2`
- [ ] Converted percentages to numbers (not strings)
- [ ] Wrapped chart in a Card or container

That's it! You're ready to use the StackedBarWithLine chart! 🎉
