# Creating a Developer-Friendly TypeScript Library Prompt

## Goal
Create a TypeScript library that's IDE-friendly, self-documenting, and intuitive to use with minimal reference to external documentation.

We should work with the existing file structure rather than creating entirely new files. Let's modify our approach to enhance the existing files with better type definitions, JSDoc comments, and developer-friendly features.

## 1. Type Definitions and JSDoc

Prioritize comprehensive TypeScript types and JSDoc comments:

```typescript
/**
 * Connects to the vehicle's ECU via Bluetooth.
 *
 * @example
 * ```typescript
 * // Simple connection
 * const result = await connectToECU();
 * 
 * // With custom options
 * const result = await connectToECU({
 *   timeout: 10000,
 *   protocol: 'AUTO',
 *   retryCount: 3
 * });
 * ```
 *
 * @param options - Connection configuration options
 * @returns A Promise resolving to the connection result
 * @throws {BluetoothUnavailableError} If Bluetooth is unavailable or disabled
 * @throws {ConnectionTimeoutError} If the connection times out
 * @throws {DeviceNotFoundError} If no compatible device is found
 * @category ECU
 */
export async function connectToECU(options?: ConnectionOptions): Promise<ConnectionResult> {
  // Implementation
}
```

## 2. Consistent Interface Patterns

Use consistent naming and parameter patterns:

```typescript
// Use descriptive, consistent naming conventions
export interface ConnectionOptions {
  /** Connection timeout in milliseconds (default: 5000) */
  timeout?: number;
  
  /** OBD protocol to use (default: 'AUTO') */
  protocol?: 'AUTO' | 'ISO9141' | 'ISO14230' | 'ISO15765' | 'SAE_J1850';
  
  /** Number of connection attempts (default: 3) */
  retryCount?: number;
}

// Use consistent return type structures
export interface ConnectionResult {
  /** Whether the connection was successful */
  success: boolean;
  
  /** Connected device information when successful */
  device?: {
    /** Device name */
    name: string;
    
    /** Device ID */
    id: string;
    
    /** Protocol used for communication */
    protocol: string;
  };
  
  /** Error information when unsuccessful */
  error?: {
    /** Error code */
    code: string;
    
    /** Human-readable error message */
    message: string;
  };
}
```

## 3. Error Handling Strategy

Create typed error classes:

```typescript
/**
 * Base error class for all ECU communication errors
 */
export class ECUError extends Error {
  /** Error code for programmatic handling */
  code: string;

  constructor(message: string, code: string) {
    super(message);
    this.name = 'ECUError';
    this.code = code;
  }
}

/**
 * Error thrown when Bluetooth is unavailable or disabled
 */
export class BluetoothUnavailableError extends ECUError {
  constructor(message = 'Bluetooth is unavailable or disabled') {
    super(message, 'ERR_BLUETOOTH_UNAVAILABLE');
    this.name = 'BluetoothUnavailableError';
  }
}
```

## 4. React Hook Design

Create hooks with consistent state management patterns:

```typescript
/**
 * Hook for managing ECU connection state
 *
 * @example
 * ```tsx
 * const VehicleConnectionScreen = () => {
 *   const { 
 *     isConnected, 
 *     isConnecting, 
 *     error, 
 *     connect, 
 *     disconnect 
 *   } = useECUConnection();
 *
 *   return (
 *     <View>
 *       {isConnecting ? (
 *         <ActivityIndicator />
 *       ) : (
 *         <Button
 *           title={isConnected ? "Disconnect" : "Connect"}
 *           onPress={isConnected ? disconnect : connect}
 *         />
 *       )}
 *       {error && <Text style={styles.error}>{error.message}</Text>}
 *     </View>
 *   );
 * };
 * ```
 *
 * @param options - Optional connection configuration
 * @returns Connection state and control functions
 */
export function useECUConnection(options?: ConnectionOptions) {
  const [isConnected, setIsConnected] = useState(false);
  const [isConnecting, setIsConnecting] = useState(false);
  const [error, setError] = useState<ECUError | null>(null);

  // Implementation details...

  return {
    isConnected,
    isConnecting,
    error,
    connect: async () => { /* ... */ },
    disconnect: async () => { /* ... */ },
  };
}
```

## 5. Include Usage Examples

Provide example files in your source code:

```typescript
import React from 'react';
import { View, Button, Text } from 'react-native';
import { useECUConnection } from '../hooks/useECUConnection';

/**
 * Example component showing a basic ECU connection flow
 */
export const BasicConnectionExample: React.FC = () => {
  const { isConnected, isConnecting, error, connect, disconnect } = useECUConnection();

  return (
    <View>
      <Text>Connection Status: {isConnected ? 'Connected' : 'Disconnected'}</Text>
      <Button
        title={isConnected ? 'Disconnect' : 'Connect to ECU'}
        onPress={isConnected ? disconnect : connect}
        disabled={isConnecting}
      />
      {isConnecting && <Text>Connecting...</Text>}
      {error && <Text>Error: {error.message}</Text>}
    </View>
  );
};
```

## 6. Provider Pattern for Global State

Make complex state management easy:

```typescript
/**
 * Provider component for ECU communication
 *
 * @example
 * ```tsx
 * // App.tsx
 * import React from 'react';
 * import { ECUProvider } from 'your-library';
 * import { MainApp } from './MainApp';
 *
 * export default function App() {
 *   return (
 *     <ECUProvider>
 *       <MainApp />
 *     </ECUProvider>
 *   );
 * }
 * ```
 */
export const ECUProvider: React.FC<{
  /** Initial connection options */
  initialOptions?: ConnectionOptions;
  
  /** Whether to automatically connect when provider mounts */
  autoConnect?: boolean;
  
  /** Children components that will have access to ECU context */
  children: React.ReactNode;
}> = ({ initialOptions, autoConnect = false, children }) => {
  // Provider implementation...
};
```

## 7. TypeScript Configuration

Use a strict TypeScript configuration:

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "es2018",
    "module": "esnext",
    "lib": ["es2018"],
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "outDir": "./lib",
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictPropertyInitialization": true,
    "noImplicitThis": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "moduleResolution": "node",
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src"],
  "exclude": ["node_modules", "**/__tests__/*"]
}
```

## 8. Use Type Exports for Type Inference

Export types for better IDE integration:

```typescript
// Export all public types from a central location
export type { ConnectionOptions } from './types/connection';
export type { DTCCode, DTCResponse } from './types/diagnostics';
export type { VehicleInfo, VINInfo } from './types/vehicle';
```

## 9. Factory Functions for Complex Objects

Create factory functions with sensible defaults:

```typescript
/**
 * Creates connection options with type safety and defaults
 */
export function createConnectionOptions({
  timeout = 5000,
  protocol = 'AUTO',
  retryCount = 3,
}: Partial<ConnectionOptions> = {}): ConnectionOptions {
  return { timeout, protocol, retryCount };
}
```

## 10. Debugging Support

Include debugging utilities:

```typescript
/**
 * Creates a logger pre-configured for ECU communications
 */
export function createECULogger(options?: LoggerOptions) {
  // Implementation...
  return {
    debug: (message: string, context?: Record<string, unknown>) => { /* ... */ },
    info: (message: string, context?: Record<string, unknown>) => { /* ... */ },
    warn: (message: string, context?: Record<string, unknown>) => { /* ... */ },
    error: (message: string, context?: Record<string, unknown>) => { /* ... */ },
  };
}
```

## Implementation Strategy

1. Start with core interfaces and types
2. Add comprehensive JSDoc to all public APIs
3. Create logical module structure
4. Implement React hooks and providers
5. Add examples for each major feature
6. Include extensive error handling
7. Set up type exports for IDE support
8. Test with TypeScript's strict mode
9. Verify auto-completion in popular IDEs

This approach will create a library that's intuitive to use directly in an IDE with excellent auto-completion and inline documentation, minimizing the need for users to reference external documentation.
