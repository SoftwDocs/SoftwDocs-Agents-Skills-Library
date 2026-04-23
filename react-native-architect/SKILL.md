---
name: react-native-architect
description: Complete React Native architecture for cross-platform mobile development. Covers navigation, state management, native modules, performance optimization, and platform-specific implementations.
tags: [react-native, mobile, ios, android, cross-platform, architecture]
version: 1.0.0
author: SoftwDocs
---

# React Native Architect

## Overview

A comprehensive skill for building production-grade React Native applications. Covers architectural patterns, navigation systems, state management, native modules, performance optimization, and platform-specific implementations for iOS and Android.

## When to Use This Skill

- Building cross-platform mobile applications
- Migrating from web to mobile
- Creating native-feeling mobile experiences
- Implementing complex navigation patterns
- Building apps with offline capabilities
- Developing apps requiring native device features

## Core Technologies

### Navigation
- React Navigation v6 (Stack, Tab, Drawer, Native)
- Deep linking support
- Navigation state persistence

### State Management
- Redux Toolkit for global state
- React Query for server state
- Zustand for local state
- Context API for theme/settings

### Performance
- Hermes engine
- Reanimated 3 for animations
- FlashList for lists
- Code splitting

## Project Structure

```
src/
├── components/
│   ├── common/          # Reusable components
│   ├── screens/         # Screen components
│   └── navigation/      # Navigation components
├── hooks/
│   ├── useAuth.ts       # Custom hooks
│   ├── useApi.ts
│   └── useTheme.ts
├── store/
│   ├── index.ts         # Store configuration
│   ├── slices/          # Redux slices
│   └── hooks.ts         # Typed hooks
├── services/
│   ├── api/             # API services
│   ├── storage/         # Local storage
│   └── navigation/      # Navigation service
├── utils/
│   ├── helpers.ts
│   ├── constants.ts
│   └── types.ts
├── assets/
│   ├── images/
│   ├── fonts/
│   └── icons/
└── theme/
    ├── colors.ts
    ├── typography.ts
    └── spacing.ts
```

## Implementation Patterns

### Pattern 1: Navigation Architecture

Type-safe navigation with React Navigation:

```typescript
// navigation/types.ts
export type RootStackParamList = {
  Home: undefined;
  Profile: { userId: string };
  Settings: undefined;
  Details: { itemId: string; from?: string };
};

export type TabParamList = {
  Home: undefined;
  Search: undefined;
  Profile: undefined;
};

export type DrawerParamList = {
  Home: undefined;
  Settings: undefined;
  About: undefined;
};

// navigation/RootNavigator.tsx
import { createNativeStackNavigator } from '@react-navigation/native-stack';
import type { RootStackParamList } from './types';

const Stack = createNativeStackNavigator<RootStackParamList>();

export function RootNavigator() {
  return (
    <Stack.Navigator
      screenOptions={{
        headerShown: true,
        headerStyle: { backgroundColor: theme.colors.primary },
        headerTintColor: theme.colors.white,
      }}
    >
      <Stack.Screen 
        name="Home" 
        component={HomeScreen}
        options={{ title: 'Home' }}
      />
      <Stack.Screen
        name="Profile"
        component={ProfileScreen}
        options={({ route }) => ({ title: `Profile ${route.params.userId}` })}
      />
      <Stack.Screen
        name="Details"
        component={DetailsScreen}
        options={({ route }) => ({ title: `Item ${route.params.itemId}` })}
      />
    </Stack.Navigator>
  );
}

// navigation/TabNavigator.tsx
import { createBottomTabNavigator } from '@react-navigation/bottom-tabs';
import type { TabParamList } from './types';

const Tab = createBottomTabNavigator<TabParamList>();

export function TabNavigator() {
  return (
    <Tab.Navigator
      screenOptions={{
        tabBarActiveTintColor: theme.colors.primary,
        tabBarInactiveTintColor: theme.colors.gray,
        headerShown: false,
      }}
    >
      <Tab.Screen
        name="Home"
        component={HomeScreen}
        options={{
          tabBarIcon: ({ color, size }) => (
            <HomeIcon color={color} size={size} />
          ),
        }}
      />
      <Tab.Screen
        name="Search"
        component={SearchScreen}
        options={{
          tabBarIcon: ({ color, size }) => (
            <SearchIcon color={color} size={size} />
          ),
        }}
      />
      <Tab.Screen
        name="Profile"
        component={ProfileScreen}
        options={{
          tabBarIcon: ({ color, size }) => (
            <ProfileIcon color={color} size={size} />
          ),
        }}
      />
    </Tab.Navigator>
  );
}

// hooks/useNavigation.ts
import { useNavigation as useReactNavigation } from '@react-navigation/native';
import type { RootStackParamList } from '../navigation/types';

export function useNavigation() {
  const navigation = useReactNavigation<RootStackParamList>();
  return navigation;
}

// Usage in components
function HomeScreen() {
  const navigation = useNavigation();
  
  const navigateToProfile = (userId: string) => {
    navigation.navigate('Profile', { userId });
  };
  
  return (
    <View>
      <Button onPress={() => navigateToProfile('123')}>
        Go to Profile
      </Button>
    </View>
  );
}
```

### Pattern 2: State Management with Redux Toolkit

Typed Redux setup with Redux Toolkit:

```typescript
// store/slices/authSlice.ts
import { createSlice, createAsyncThunk, PayloadAction } from '@reduxjs/toolkit';
import { api } from '../../services/api';

interface AuthState {
  user: User | null;
  token: string | null;
  isLoading: boolean;
  error: string | null;
}

const initialState: AuthState = {
  user: null,
  token: null,
  isLoading: false,
  error: null,
};

export const login = createAsyncThunk(
  'auth/login',
  async (credentials: { email: string; password: string }) => {
    const response = await api.post('/auth/login', credentials);
    return response.data;
  }
);

export const logout = createAsyncThunk('auth/logout', async () => {
  await api.post('/auth/logout');
});

const authSlice = createSlice({
  name: 'auth',
  initialState,
  reducers: {
    clearError: (state) => {
      state.error = null;
    },
    setUser: (state, action: PayloadAction<User>) => {
      state.user = action.payload;
    },
  },
  extraReducers: (builder) => {
    builder
      .addCase(login.pending, (state) => {
        state.isLoading = true;
        state.error = null;
      })
      .addCase(login.fulfilled, (state, action) => {
        state.isLoading = false;
        state.user = action.payload.user;
        state.token = action.payload.token;
      })
      .addCase(login.rejected, (state, action) => {
        state.isLoading = false;
        state.error = action.error.message || 'Login failed';
      })
      .addCase(logout.fulfilled, (state) => {
        state.user = null;
        state.token = null;
      });
  },
});

export const { clearError, setUser } = authSlice.actions;
export default authSlice.reducer;

// store/index.ts
import { configureStore } from '@reduxjs/toolkit';
import authReducer from './slices/authSlice';
import { api } from '../services/api';

export const store = configureStore({
  reducer: {
    auth: authReducer,
    [api.reducerPath]: api.reducer,
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(api.middleware),
});

export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;

// store/hooks.ts
import { useDispatch, useSelector } from 'react-redux';
import type { RootState, AppDispatch } from './index';

export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector = <T>(selector: (state: RootState) => T): T => {
  return useSelector(selector);
};

// Usage in components
function LoginScreen() {
  const dispatch = useAppDispatch();
  const { isLoading, error, user } = useAppSelector((state) => state.auth);
  
  const handleLogin = async (email: string, password: string) => {
    try {
      await dispatch(login({ email, password })).unwrap();
    } catch (err) {
      console.error('Login failed', err);
    }
  };
  
  return (
    <View>
      {isLoading && <ActivityIndicator />}
      {error && <Text>{error}</Text>}
      <TextInput placeholder="Email" />
      <TextInput placeholder="Password" secureTextEntry />
      <Button onPress={() => handleLogin('test@test.com', 'password')}>
        Login
      </Button>
    </View>
  );
}
```

### Pattern 3: API Integration with React Query

Type-safe API calls with React Query:

```typescript
// services/api.ts
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';
import type { RootState } from '../store';

export const api = createApi({
  reducerPath: 'api',
  baseQuery: fetchBaseQuery({
    baseUrl: API_BASE_URL,
    prepareHeaders: (headers, { getState }) => {
      const token = (getState() as RootState).auth.token;
      if (token) {
        headers.set('authorization', `Bearer ${token}`);
      }
      return headers;
    },
  }),
  tagTypes: ['User', 'Post', 'Comment'],
  endpoints: (builder) => ({
    getUsers: builder.query<User[], void>({
      query: () => '/users',
      providesTags: ['User'],
    }),
    getUser: builder.query<User, string>({
      query: (id) => `/users/${id}`,
      providesTags: (result, error, id) => [{ type: 'User', id }],
    }),
    createUser: builder.mutation<User, Partial<User>>({
      query: (body) => ({
        url: '/users',
        method: 'POST',
        body,
      }),
      invalidatesTags: ['User'],
    }),
    updateUser: builder.mutation<User, { id: string; data: Partial<User> }>({
      query: ({ id, data }) => ({
        url: `/users/${id}`,
        method: 'PUT',
        body: data,
      }),
      invalidatesTags: (result, error, { id }) => [{ type: 'User', id }],
    }),
  }),
});

export const { useGetUsersQuery, useGetUserQuery, useCreateUserMutation, useUpdateUserMutation } = api;

// hooks/useApi.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';
import { api } from '../services/api';

export function useUsers() {
  return useQuery({
    queryKey: ['users'],
    queryFn: () => api.getUsers().unwrap(),
  });
}

export function useUser(id: string) {
  return useQuery({
    queryKey: ['user', id],
    queryFn: () => api.getUser(id).unwrap(),
    enabled: !!id,
  });
}

export function useCreateUser() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: (data: Partial<User>) => api.createUser(data).unwrap(),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] });
    },
  });
}

// Usage in components
function UserListScreen() {
  const { data: users, isLoading, error } = useUsers();
  const createUser = useCreateUser();
  
  const handleCreateUser = () => {
    createUser.mutate({ name: 'New User', email: 'new@test.com' });
  };
  
  if (isLoading) return <ActivityIndicator />;
  if (error) return <Text>Error loading users</Text>;
  
  return (
    <FlatList
      data={users}
      renderItem={({ item }) => <UserItem user={item} />}
      ListHeaderComponent={
        <Button onPress={handleCreateUser}>Create User</Button>
      }
    />
  );
}
```

### Pattern 4: Native Modules

Creating custom native modules for iOS and Android:

```typescript
// ios/CustomModule.h
#import <React/RCTBridgeModule.h>

@interface RCT_EXTERN_MODULE(CustomModule, NSObject)

RCT_EXTERN_METHOD(getDeviceInfo:(RCTPromiseResolveBlock)resolve
                  reject:(RCTPromiseRejectBlock)reject)

RCT_EXTERN_METHOD(openSettings:(RCTPromiseResolveBlock)resolve
                  reject:(RCTPromiseRejectBlock)reject)

@end

// ios/CustomModule.m
#import "CustomModule.h"
#import <UIKit/UIKit.h>

@implementation CustomModule

- (dispatch_queue_t)methodQueue {
  return dispatch_get_main_queue();
}

RCT_EXPORT_METHOD();

- (void)getDeviceInfo:(RCTPromiseResolveBlock)resolve
                  reject:(RCTPromiseRejectBlock)reject {
  UIDevice *device = [UIDevice currentDevice];
  NSDictionary *info = @{
    @"model": device.model,
    @"systemName": device.systemName,
    @"systemVersion": device.systemVersion,
    @"appVersion": [[NSBundle mainBundle] objectForInfoDictionaryKey:@"CFBundleShortVersionString"],
  };
  resolve(info);
}

- (void)openSettings:(RCTPromiseResolveBlock)resolve
                  reject:(RCTPromiseRejectBlock)reject {
  NSURL *url = [NSURL URLWithString:UIApplicationOpenSettingsURLString];
  if ([[UIApplication sharedApplication] canOpenURL:url]) {
    [[UIApplication sharedApplication] openURL:url options:@{} completionHandler:^(BOOL success) {
      resolve(@(success));
    }];
  } else {
    reject(@"settings_unavailable", @"Settings not available", nil);
  }
}

@end

// android/app/src/main/java/com/custommodule/CustomModule.java
package com.custommodule;

import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.bridge.ReactContextBaseJavaModule;
import com.facebook.react.bridge.ReactMethod;
import com.facebook.react.bridge.Promise;
import com.facebook.react.bridge.WritableMap;
import com.facebook.react.bridge.WritableNativeMap;
import android.content.Context;
import android.os.Build;
import android.provider.Settings;

public class CustomModule extends ReactContextBaseJavaModule {
  private static final String E_UNAVAILABLE = "E_UNAVAILABLE";

  public CustomModule(ReactApplicationContext reactContext) {
    super(reactContext);
  }

  @Override
  public String getName() {
    return "CustomModule";
  }

  @ReactMethod
  public void getDeviceInfo(Promise promise) {
    try {
      WritableMap info = new WritableNativeMap();
      info.putString("model", Build.MODEL);
      info.putString("manufacturer", Build.MANUFACTURER);
      info.putString("version", Build.VERSION.RELEASE);
      info.putString("appVersion", getCurrentActivity()
        .getPackageManager()
        .getPackageInfo(getCurrentActivity().getPackageName(), 0)
        .versionName);
      promise.resolve(info);
    } catch (Exception e) {
      promise.reject(E_UNAVAILABLE, e.getMessage());
    }
  }

  @ReactMethod
  public void openSettings(Promise promise) {
    try {
      Intent intent = new Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS);
      intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
      getCurrentActivity().startActivity(intent);
      promise.resolve(true);
    } catch (Exception e) {
      promise.reject(E_UNAVAILABLE, e.getMessage());
    }
  }
}

// android/app/src/main/java/com/custommodule/CustomModulePackage.java
package com.custommodule;

import com.facebook.react.ReactPackage;
import com.facebook.react.bridge.NativeModule;
import com.facebook.react.bridge.ReactApplicationContext;
import com.facebook.react.uimanager.ViewManager;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class CustomModulePackage implements ReactPackage {
  @Override
  public List<ViewManager> createViewManagers(ReactApplicationContext reactContext) {
    return Collections.emptyList();
  }

  @Override
  public List<NativeModule> createNativeModules(ReactApplicationContext reactContext) {
    List<NativeModule> modules = new ArrayList<>();
    modules.add(new CustomModule(reactContext));
    return modules;
  }
}

// TypeScript interface
import { NativeModules } from 'react-native';

interface CustomModuleInterface {
  getDeviceInfo(): Promise<DeviceInfo>;
  openSettings(): Promise<boolean>;
}

const CustomModule = NativeModules.CustomModule as CustomModuleInterface;

export default CustomModule;

// Usage
function DeviceInfoScreen() {
  const [deviceInfo, setDeviceInfo] = useState<DeviceInfo | null>(null);
  
  useEffect(() => {
    CustomModule.getDeviceInfo().then(setDeviceInfo);
  }, []);
  
  return (
    <View>
      <Text>Model: {deviceInfo?.model}</Text>
      <Text>Version: {deviceInfo?.version}</Text>
      <Button onPress={() => CustomModule.openSettings()}>
        Open Settings
      </Button>
    </View>
  );
}
```

### Pattern 5: Performance Optimization

Optimizing React Native app performance:

```typescript
// Using FlashList for efficient lists
import { FlashList } from '@shopify/flash-list';

function OptimizedList({ items }: { items: Item[] }) {
  return (
    <FlashList
      data={items}
      estimatedItemSize={100}
      renderItem={({ item }) => <ListItem item={item} />}
      keyExtractor={(item) => item.id}
      numColumns={2}
      horizontal={false}
    />
  );
}

// Memoizing components
const ListItem = React.memo(({ item }: { item: Item }) => {
  return (
    <View style={styles.item}>
      <Text>{item.name}</Text>
    </View>
  );
});

// Using useCallback for event handlers
function ParentComponent() {
  const handlePress = useCallback((id: string) => {
    console.log('Pressed', id);
  }, []);
  
  return <ChildComponent onPress={handlePress} />;
}

// Code splitting with React Native Navigation
import { useNavigation } from '@react-navigation/native';

function LazyScreen() {
  const navigation = useNavigation();
  
  useEffect(() => {
    // Lazy load screen when needed
    import('./HeavyScreen').then((module) => {
      navigation.setOptions({
        headerRight: () => <module.HeaderComponent />,
      });
    });
  }, [navigation]);
  
  return <View>Content</View>;
}

// Optimizing images
import FastImage from 'react-native-fast-image';

function OptimizedImage({ uri }: { uri: string }) {
  return (
    <FastImage
      source={{ uri, priority: FastImage.priority.high }}
      resizeMode={FastImage.resizeMode.cover}
      style={styles.image}
    />
  );
}

// Using Hermes
// In metro.config.js
module.exports = {
  transformer: {
    getTransformOptions: async () => ({
      transform: {
        experimentalImportSupport: false,
        inlineRequires: false,
      },
    }),
  },
};

// Using Reanimated for smooth animations
import Animated, {
  useSharedValue,
  useAnimatedStyle,
  withSpring,
} from 'react-native-reanimated';

function AnimatedComponent() {
  const scale = useSharedValue(1);
  
  const animatedStyle = useAnimatedStyle(() => ({
    transform: [{ scale: scale.value }],
  }));
  
  const handlePress = () => {
    scale.value = withSpring(scale.value === 1 ? 1.2 : 1);
  };
  
  return (
    <Animated.View style={[styles.box, animatedStyle]}>
      <Button onPress={handlePress}>Animate</Button>
    </Animated.View>
  );
}
```

## Platform-Specific Code

```typescript
// Platform-specific implementations
import { Platform, PlatformIOS } from 'react-native';

function PlatformSpecificComponent() {
  if (Platform.OS === 'ios') {
    return <IOSComponent />;
  }
  return <AndroidComponent />;
}

// Platform-specific styles
const styles = StyleSheet.create({
  container: {
    ...Platform.select({
      ios: {
        shadowColor: '#000',
        shadowOffset: { width: 0, height: 2 },
        shadowOpacity: 0.25,
        shadowRadius: 3.84,
      },
      android: {
        elevation: 5,
      },
    }),
  },
});

// Platform-specific files
// Component.ios.tsx
export function Component() {
  return <IOSView />;
}

// Component.android.tsx
export function Component() {
  return <AndroidView />;
}
```

## Best Practices

### ✅ Do

- Use TypeScript for type safety
- Implement proper error boundaries
- Use memoization for performance
- Test on both iOS and Android
- Follow platform design guidelines
- Use proper navigation patterns
- Implement offline support
- Use proper state management

### ❌ Don't

- Use `any` types
- Ignore platform differences
- Over-optimize prematurely
- Skip testing on real devices
- Use inline styles extensively
- Forget to clean up subscriptions
- Ignore memory leaks
- Use synchronous operations on main thread

## Resources

- [React Native Documentation](https://reactnative.dev/)
- [React Navigation](https://reactnavigation.org/)
- [Redux Toolkit](https://redux-toolkit.js.org/)
- [React Query](https://tanstack.com/query/latest)
- [Reanimated](https://docs.swmansion.com/react-native-reanimated/)
