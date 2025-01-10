import pandas as pd
import numpy as np
import timeit
import random

# Завантаження даних
# Завантажте дані з файлу або використовуйте тестовий набір даних
# data = pd.read_csv('household_power_consumption.txt', sep=';', na_values='?', low_memory=False)
data = pd.DataFrame({
    'date': pd.date_range(start='1/1/2022', periods=100000, freq='T'),
    'time': np.random.choice(['12:00:00', '18:30:00', '22:00:00'], 100000),
    'global_active_power': np.random.uniform(1.0, 10.0, 100000),
    'voltage': np.random.uniform(220.0, 250.0, 100000),
    'global_intensity': np.random.uniform(10.0, 25.0, 100000),
    'sub_metering_1': np.random.uniform(0.0, 10.0, 100000),
    'sub_metering_2': np.random.uniform(0.0, 10.0, 100000),
    'sub_metering_3': np.random.uniform(0.0, 10.0, 100000)
})

# Функції для фільтрації даних
def filter_power(data, threshold=5.0):
    return data[data['global_active_power'] > threshold]

def filter_voltage(data, threshold=235.0):
    return data[data['voltage'] > threshold]

def filter_current_and_compare(data, min_current=19.0, max_current=20.0):
    filtered = data[(data['global_intensity'] >= min_current) & (data['global_intensity'] <= max_current)]
    return filtered[filtered['sub_metering_2'] > filtered['sub_metering_3']]

def random_sample(data, n=500000):
    return data.sample(n=min(n, len(data)), replace=False)

def evening_consumption(data, power_threshold=6.0):
    filtered = data[(data['global_active_power'] > power_threshold) & (data['time'] > '18:00:00')]
    first_half = filtered.iloc[:len(filtered)//2:3]
    second_half = filtered.iloc[len(filtered)//2::4]
    return pd.concat([first_half, second_half])

# Профілювання часу виконання
print("Time profiling for numpy and pandas")

# Використання timeit для різних функцій
data_numpy = data.to_numpy()
data_pandas = data

print("Filter by global_active_power using numpy:", 
      timeit.timeit(lambda: data_numpy[data_numpy[:, 2].astype(float) > 5], number=10))
print("Filter by global_active_power using pandas:", 
      timeit.timeit(lambda: filter_power(data_pandas), number=10))

# Аналіз результатів
power_filtered = filter_power(data)
voltage_filtered = filter_voltage(data)
current_filtered = filter_current_and_compare(data)
sampled_data = random_sample(data, 5000)
evening_filtered = evening_consumption(data)

# Висновки
print("Фільтр за потужністю:", len(power_filtered))
print("Фільтр за напругою:", len(voltage_filtered))
print("Фільтр за силою струму:", len(current_filtered))
print("Випадкова вибірка:", len(sampled_data))
print("Споживання ввечері:", len(evening_filtered))
