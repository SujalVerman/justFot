# StackedBarWithLine Chart Usage Documentation

## Overview
This document explains how the `StackedBarWithLine1` component is used in the `opt.tsx` page. This chart component combines stacked bar charts with a line chart overlay, allowing visualization of multiple metrics simultaneously.

## Component Import

```typescript
import StackedBarWithLine1 from '@/components/mf/charts/StackedBarwithLine1';
```

## Component Location
- **Component File**: `src/components/mf/charts/StackedBarwithLine1.tsx`
- **Usage File**: `opt.tsx`

## Usage Examples in opt.tsx

The `StackedBarWithLine1` component is used in **two places** in the `opt.tsx` file:

### 1. Date Wise Trend Chart

**Location**: Lines 1307-1336

```typescript
<StackedBarWithLine1
  chartData={dwTrendChartData}
  chartConfig={dwTrendChartConfig}
  isHorizontal={false}
  isLegend={true}
  isLoading={dwTrendLoading}
  selectoptions={dwTrendSelectOptions}
  selectedFrequency={dwTrendSelectedFrequency}
  handleFrequencyChange={handleDwTrendFrequencyChange}
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
```

### 2. Publisher/Vendor Wise Trend Chart

**Location**: Lines 1360-1386

```typescript
<StackedBarWithLine1
  chartData={publisherVendorChartData}
  chartConfig={publisherVendorChartConfig}
  isHorizontal={false}
  isLegend={true}
  isLoading={publisherVendorLoading}
  xAxisConfig={{
    dataKey: "name",
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
```

## Data Preparation

### Date Wise Trend Chart Data

**State Declaration** (Line 999):
```typescript
const [dwTrendChartData, setDwTrendChartData] = useState<any[]>([]);
```

**Chart Configuration** (Lines 1001-1005):
```typescript
const dwTrendChartConfig = {
  "Total Count": { label: "Valid Count", color: "#00A86B" },
  "Fraud Count": { label: "Invalid Count", color: "#FF0000" },
  "Fraud %": { label: "Invalid %", color: "#b91c1c" },
};
```

**Data Mapping** (Lines 1016-1040):
The API response is mapped based on the selected frequency (Daily, Weekly, Monthly):

- **Monthly**: Maps `month`, `total_count`, `fraud_count`, and `fraud_percentage`
- **Weekly**: Maps `week`, `total_count`, `fraud_count`, and `fraud_percentage`
- **Daily**: Maps `date`, calculates total count from `clean_count + fraud_count`, `fraud_count`, and `fraud_percentage`

**API Call** (Lines 1012-1045):
```typescript
const { result: dwTrendApiResult, loading: dwTrendLoading } = useApiCall<any[]>({
  url: `https://uat-api-dev.mfilterit.net/v1/app/${selectedType}/trends`,
  method: "POST",
  manual: true,
  onSuccess: (response) => {
    // Data mapping logic here
    setDwTrendChartData(mappedData);
  },
});
```

**Frequency Selection** (Lines 998, 1000, 1057-1059):
- State: `dwTrendSelectedFrequency` (default: "Daily")
- Options: `["Daily", "Weekly", "Monthly"]`
- Handler: `handleDwTrendFrequencyChange`

### Publisher/Vendor Wise Trend Chart Data

**State Declaration** (Line 1062):
```typescript
const [publisherVendorChartData, setPublisherVendorChartData] = useState<any[]>([]);
```

**Chart Configuration** (Lines 1063-1067):
```typescript
const publisherVendorChartConfig = {
  "Clean Count": { label: "Valid Count", color: "#00A86B" },
  "Fraud Count": { label: "Invalid Count", color: "#FF0000" },
  "Fraud %": { label: "Invalid %", color: "#b91c1c" },
};
```

**Data Mapping** (Lines 1073-1088):
The API response is mapped based on the selected radio button (Publisher or Vendor):

```typescript
const mapped = response.map((item: any) => {
  const isPublisher = selectedRadio === 'Publisher';
  const nameKey = isPublisher ? 'publisher_name' : 'agency_name';
  return {
    name: item[nameKey],
    "Clean Count": item.clean_count || 0,
    "Fraud Count": item.fraud_count || 0,
    "Fraud %": parseFloat((item.fraud_percentage || '').replace("%", "")),
  };
});
```

**API Call** (Lines 1069-1093):
```typescript
const { result: publisherVendorApiResult, loading: publisherVendorLoading } = useApiCall<any[]>({
  url: `https://uat-api-dev.mfilterit.net/v1/app/${selectedType}/publisher_trends`,
  method: "POST",
  manual: true,
  onSuccess: (response) => {
    // Data mapping logic here
    setPublisherVendorChartData(mapped);
  },
});
```

## Component Props

### Required Props

| Prop | Type | Description |
|------|------|-------------|
| `chartData` | `TrafficTrendData[]` | Array of data objects for the chart |
| `chartConfig` | `Object` | Configuration object mapping data keys to labels and colors |
| `onExpand` | `() => void` | Callback function for expand action |

### Optional Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `isLoading` | `boolean` | `false` | Shows loading spinner when true |
| `isLegend` | `boolean` | `true` | Shows/hides the legend |
| `isHorizontal` | `boolean` | `false` | Orientation of the chart |
| `title` | `string` | - | Chart title |
| `onExport` | `(s: string, title: string, index: number) => void` | - | Export callback |
| `rightAxisType` | `"percentage" \| "price" \| "number"` | `"percentage"` | Format for right Y-axis |
| `height` | `number` | `180` | Chart height in pixels |
| `selectoptions` | `string[]` | - | Options for frequency selector |
| `selectedFrequency` | `string` | - | Currently selected frequency |
| `handleFrequencyChange` | `(value: string) => void` | - | Handler for frequency change |
| `xAxisConfig` | `Object` | - | X-axis configuration object |
| `YAxis1` | `Object` | - | Left Y-axis configuration |
| `YAxis2` | `Object` | - | Right Y-axis configuration |

### Chart Config Structure

```typescript
{
  "Data Key": {
    label: "Display Label",
    color: "#HEXCOLOR"
  }
}
```

**Example**:
```typescript
{
  "Total Count": { label: "Valid Count", color: "#00A86B" },
  "Fraud Count": { label: "Invalid Count", color: "#FF0000" },
  "Fraud %": { label: "Invalid %", color: "#b91c1c" },
}
```

### Chart Data Structure

```typescript
[
  {
    month: "Jan",           // X-axis value
    "Total Count": 100,     // Stacked bar value 1
    "Fraud Count": 20,      // Stacked bar value 2
    "Fraud %": 20.0         // Line chart value (right Y-axis)
  },
  // ... more data points
]
```

## Axis Configuration

### X-Axis Config

```typescript
xAxisConfig={{
  dataKey: "month",        // Key from data object for X-axis
  tickLine: false,         // Show/hide tick lines
  tickMargin: 10,          // Margin for ticks
  axisLine: false,         // Show/hide axis line
  tickFormatter: (value: string) => value,  // Format tick labels
  textAnchor: "middle",    // Text alignment
  dy: 10,                  // Vertical offset
}}
```

### Y-Axis Configuration

**Left Y-Axis** (for stacked bars):
```typescript
YAxis1={{
  yAxisId: "left",
  orientation: "left",
  stroke: "hsl(var(--chart-1))",
}}
```

**Right Y-Axis** (for line chart):
```typescript
YAxis2={{
  yAxisId: "right",
  orientation: "right",
  stroke: "hsl(var(--chart-3))",
}}
```

## How It Works

1. **Stacked Bars**: Display multiple metrics stacked on top of each other (e.g., "Total Count" and "Fraud Count")
2. **Line Chart**: Overlays a line chart on the same graph (e.g., "Fraud %")
3. **Dual Y-Axes**: 
   - Left Y-axis: For stacked bar values (counts)
   - Right Y-axis: For line chart values (percentages, prices, or numbers)
4. **Responsive**: Automatically adjusts width based on data points
5. **Interactive**: Supports tooltips, legends, and frequency selection

## Key Features

- ✅ Combines stacked bar chart with line chart
- ✅ Dual Y-axes (left for bars, right for line)
- ✅ Customizable colors and labels
- ✅ Loading states
- ✅ Frequency selection (Daily/Weekly/Monthly)
- ✅ Responsive design
- ✅ Legend support
- ✅ Custom axis formatting

## Dependencies

- **Recharts**: `Bar`, `CartesianGrid`, `XAxis`, `YAxis`, `Line`, `ComposedChart`, `ResponsiveContainer`
- **UI Components**: `Card`, `CardContent`, `ChartContainer`, `ChartTooltip`
- **Utils**: `getXAxisAngle`, `formatNumber` from `@/lib/utils`

## Notes

- The component uses `ComposedChart` from Recharts to combine bars and lines
- The line chart typically represents percentage values on the right Y-axis
- Stacked bars represent count values on the left Y-axis
- Data is automatically cleaned and formatted (percentages converted to numbers, prices parsed)
- The chart width adjusts dynamically based on the number of data points

