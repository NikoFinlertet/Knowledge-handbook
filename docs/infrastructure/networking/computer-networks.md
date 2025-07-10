# Компьютерные сети и Интернет

> Фундаментальные знания о сетях ЭВМ, протоколах и современных сетевых технологиях

## Содержание

### 🌐 Основы сетей
1. [Модель OSI](#модель-osi)
2. [Стек протоколов TCP/IP](#стек-протоколов-tcpip)
3. [Типы сетей](#типы-сетей)

### 🔌 Физический уровень
4. [Среды передачи данных](#среды-передачи-данных)
5. [Методы кодирования](#методы-кодирования)
6. [Топологии сетей](#топологии-сетей)

### 🔗 Канальный уровень
7. [Протокол Ethernet](#протокол-ethernet)
8. [Коммутация](#коммутация)
9. [VLAN](#vlan)

### 🌍 Сетевой уровень
10. [Протокол IP](#протокол-ip)
11. [Маршрутизация](#маршрутизация)
12. [IPv6](#ipv6)

### 🚚 Транспортный уровень
13. [Протокол TCP](#протокол-tcp)
14. [Протокол UDP](#протокол-udp)
15. [Управление потоком](#управление-потоком)

### 📡 Прикладной уровень
16. [HTTP/HTTPS](#httphttps)
17. [DNS](#dns)
18. [SMTP/POP3/IMAP](#smtppop3imap)

### 🔒 Безопасность сетей
19. [Сетевая безопасность](#сетевая-безопасность)
20. [VPN](#vpn)
21. [Брандмауэры](#брандмауэры)

### 🚀 Современные технологии
22. [SDN](#sdn)
23. [QoS](#qos)
24. [Cloud Networking](#cloud-networking)

---

## Модель OSI

### Семиуровневая модель

```python
class OSIModel:
    """Модель взаимодействия открытых систем"""
    
    def __init__(self):
        self.layers = {
            7: {
                'name': 'Application',
                'name_ru': 'Прикладной',
                'description': 'Интерфейс для пользовательских приложений',
                'protocols': ['HTTP', 'HTTPS', 'FTP', 'SMTP', 'DNS', 'DHCP'],
                'examples': ['Веб-браузеры', 'Email-клиенты', 'Файловые менеджеры'],
                'data_unit': 'Данные'
            },
            6: {
                'name': 'Presentation',
                'name_ru': 'Представления',
                'description': 'Кодирование, сжатие, шифрование данных',
                'protocols': ['SSL/TLS', 'JPEG', 'GIF', 'MPEG'],
                'examples': ['Шифрование', 'Сжатие', 'Кодировки'],
                'data_unit': 'Данные'
            },
            5: {
                'name': 'Session',
                'name_ru': 'Сеансовый',
                'description': 'Управление сеансами связи',
                'protocols': ['NetBIOS', 'RPC', 'SQL'],
                'examples': ['Установка сеанса', 'Синхронизация', 'Восстановление'],
                'data_unit': 'Данные'
            },
            4: {
                'name': 'Transport',
                'name_ru': 'Транспортный',
                'description': 'Надежная доставка данных от узла к узлу',
                'protocols': ['TCP', 'UDP', 'SCTP'],
                'examples': ['Сегментация', 'Контроль ошибок', 'Управление потоком'],
                'data_unit': 'Сегменты/Дейтаграммы'
            },
            3: {
                'name': 'Network',
                'name_ru': 'Сетевой',
                'description': 'Маршрутизация пакетов между сетями',
                'protocols': ['IP', 'ICMP', 'ARP', 'OSPF', 'BGP'],
                'examples': ['Маршрутизаторы', 'IP-адресация', 'Маршрутизация'],
                'data_unit': 'Пакеты'
            },
            2: {
                'name': 'Data Link',
                'name_ru': 'Канальный',
                'description': 'Передача данных между соседними узлами',
                'protocols': ['Ethernet', 'Wi-Fi', 'PPP', 'Frame Relay'],
                'examples': ['Коммутаторы', 'MAC-адреса', 'Обнаружение ошибок'],
                'data_unit': 'Кадры'
            },
            1: {
                'name': 'Physical',
                'name_ru': 'Физический',
                'description': 'Передача битов по физической среде',
                'protocols': ['Ethernet Physical', 'USB', 'Bluetooth'],
                'examples': ['Кабели', 'Хабы', 'Повторители', 'Радиоволны'],
                'data_unit': 'Биты'
            }
        }
    
    def get_layer_info(self, layer_number):
        """Получить информацию о слое"""
        return self.layers.get(layer_number)
    
    def data_encapsulation_demo(self):
        """Демонстрация инкапсуляции данных"""
        
        # Исходные данные
        original_data = "Hello, World!"
        
        encapsulation_process = {
            'step_7': {
                'layer': 'Application',
                'data': original_data,
                'description': 'Пользовательские данные'
            },
            'step_6': {
                'layer': 'Presentation', 
                'data': f"COMPRESSED[{original_data}]",
                'description': 'Сжатие и кодирование'
            },
            'step_5': {
                'layer': 'Session',
                'data': f"SESSION_ID:12345|COMPRESSED[{original_data}]",
                'description': 'Добавление информации о сеансе'
            },
            'step_4': {
                'layer': 'Transport',
                'data': f"TCP_HEADER|SESSION_ID:12345|COMPRESSED[{original_data}]",
                'description': 'TCP заголовок с портами и контролем'
            },
            'step_3': {
                'layer': 'Network',
                'data': f"IP_HEADER|TCP_HEADER|SESSION_ID:12345|COMPRESSED[{original_data}]",
                'description': 'IP заголовок с адресами отправителя и получателя'
            },
            'step_2': {
                'layer': 'Data Link',
                'data': f"ETH_HEADER|IP_HEADER|TCP_HEADER|SESSION_ID:12345|COMPRESSED[{original_data}]|ETH_TRAILER",
                'description': 'Ethernet заголовок и трейлер с MAC-адресами'
            },
            'step_1': {
                'layer': 'Physical',
                'data': "101010100110110101...",
                'description': 'Преобразование в биты для передачи по физической среде'
            }
        }
        
        return encapsulation_process
    
    def protocol_interaction_example(self):
        """Пример взаимодействия протоколов"""
        
        web_request_flow = {
            'scenario': 'Запрос веб-страницы',
            'layers': {
                7: {
                    'action': 'HTTP GET запрос',
                    'details': 'GET /index.html HTTP/1.1\\nHost: example.com'
                },
                6: {
                    'action': 'TLS шифрование (если HTTPS)',
                    'details': 'Шифрование HTTP данных'
                },
                5: {
                    'action': 'Установка TCP сеанса',
                    'details': 'Управление соединением'
                },
                4: {
                    'action': 'TCP сегментация',
                    'details': 'Разбиение на сегменты, добавление портов (80/443)'
                },
                3: {
                    'action': 'IP маршрутизация',
                    'details': 'Добавление IP адресов источника и назначения'
                },
                2: {
                    'action': 'Ethernet инкапсуляция',
                    'details': 'Добавление MAC адресов, создание кадра'
                },
                1: {
                    'action': 'Физическая передача',
                    'details': 'Передача битов по сетевому кабелю/WiFi'
                }
            }
        }
        
        return web_request_flow

# Демонстрация модели OSI
def demonstrate_osi_model():
    """Демонстрация работы модели OSI"""
    
    osi = OSIModel()
    
    print("=== Модель OSI ===")
    print("Семь уровней сетевого взаимодействия:\n")
    
    for layer_num in range(7, 0, -1):
        layer_info = osi.get_layer_info(layer_num)
        print(f"{layer_num}. {layer_info['name_ru']} ({layer_info['name']})")
        print(f"   Назначение: {layer_info['description']}")
        print(f"   Протоколы: {', '.join(layer_info['protocols'])}")
        print(f"   Единица данных: {layer_info['data_unit']}")
        print()
    
    print("=== Инкапсуляция данных ===")
    encapsulation = osi.data_encapsulation_demo()
    
    for step, info in encapsulation.items():
        print(f"{info['layer']}: {info['description']}")
        print(f"Данные: {info['data']}")
        print()
    
    print("=== Взаимодействие протоколов ===")
    web_flow = osi.protocol_interaction_example()
    print(f"Сценарий: {web_flow['scenario']}")
    
    for layer_num in range(7, 0, -1):
        layer_action = web_flow['layers'][layer_num]
        print(f"Уровень {layer_num}: {layer_action['action']}")
        print(f"  Детали: {layer_action['details']}")

# Запуск демонстрации
demonstrate_osi_model()
```

---

## Стек протоколов TCP/IP

### Четырехуровневая модель

```python
class TCPIPStack:
    """Стек протоколов TCP/IP"""
    
    def __init__(self):
        self.layers = {
            4: {
                'name': 'Application Layer',
                'name_ru': 'Прикладной уровень',
                'description': 'Протоколы приложений',
                'protocols': {
                    'HTTP': {'port': 80, 'description': 'Передача веб-страниц'},
                    'HTTPS': {'port': 443, 'description': 'Защищенная передача веб-страниц'},
                    'FTP': {'port': 21, 'description': 'Передача файлов'},
                    'SMTP': {'port': 25, 'description': 'Отправка электронной почты'},
                    'POP3': {'port': 110, 'description': 'Получение электронной почты'},
                    'IMAP': {'port': 143, 'description': 'Управление электронной почтой'},
                    'DNS': {'port': 53, 'description': 'Система доменных имен'},
                    'DHCP': {'port': 67, 'description': 'Автоматическое назначение IP'},
                    'SSH': {'port': 22, 'description': 'Безопасный удаленный доступ'},
                    'Telnet': {'port': 23, 'description': 'Удаленный доступ'}
                },
                'osi_equivalent': [5, 6, 7]
            },
            3: {
                'name': 'Transport Layer',
                'name_ru': 'Транспортный уровень',
                'description': 'Надежная доставка данных',
                'protocols': {
                    'TCP': {
                        'description': 'Протокол с установлением соединения',
                        'features': ['Надежность', 'Контроль потока', 'Контроль ошибок', 'Упорядочивание']
                    },
                    'UDP': {
                        'description': 'Протокол без установления соединения',
                        'features': ['Быстрота', 'Простота', 'Широковещание', 'Минимальные накладные расходы']
                    }
                },
                'osi_equivalent': [4]
            },
            2: {
                'name': 'Internet Layer',
                'name_ru': 'Межсетевой уровень',
                'description': 'Маршрутизация между сетями',
                'protocols': {
                    'IP': {'description': 'Интернет протокол', 'versions': ['IPv4', 'IPv6']},
                    'ICMP': {'description': 'Протокол контрольных сообщений'},
                    'ARP': {'description': 'Протокол разрешения адресов'},
                    'RARP': {'description': 'Обратный протокол разрешения адресов'}
                },
                'osi_equivalent': [3]
            },
            1: {
                'name': 'Network Access Layer',
                'name_ru': 'Уровень сетевого доступа',
                'description': 'Физическая передача данных',
                'protocols': {
                    'Ethernet': {'description': 'Проводные локальные сети'},
                    'Wi-Fi': {'description': 'Беспроводные локальные сети'},
                    'PPP': {'description': 'Соединение точка-точка'},
                    'Frame Relay': {'description': 'Сети с коммутацией кадров'}
                },
                'osi_equivalent': [1, 2]
            }
        }
    
    def compare_with_osi(self):
        """Сравнение TCP/IP с моделью OSI"""
        
        comparison = {
            'tcp_ip_layers': 4,
            'osi_layers': 7,
            'differences': {
                'Application Layer (TCP/IP)': 'Объединяет Прикладной, Представления и Сеансовый уровни OSI',
                'Transport Layer': 'Соответствует Транспортному уровню OSI',
                'Internet Layer': 'Соответствует Сетевому уровню OSI',
                'Network Access Layer': 'Объединяет Канальный и Физический уровни OSI'
            },
            'advantages_tcp_ip': [
                'Простота и практичность',
                'Широкое распространение',
                'Масштабируемость',
                'Открытость стандартов'
            ],
            'advantages_osi': [
                'Детализированная модель',
                'Четкое разделение функций',
                'Универсальность',
                'Образовательная ценность'
            ]
        }
        
        return comparison
    
    def packet_journey_simulation(self):
        """Симуляция путешествия пакета через стек TCP/IP"""
        
        def send_data(source_app, dest_app, data):
            """Симуляция отправки данных"""
            
            journey = []
            
            # Уровень 4 - Application
            journey.append({
                'layer': 4,
                'name': 'Application Layer',
                'action': 'Создание данных приложением',
                'details': f'{source_app} отправляет "{data}" в {dest_app}',
                'data_size': len(data)
            })
            
            # Уровень 3 - Transport  
            tcp_header_size = 20
            journey.append({
                'layer': 3,
                'name': 'Transport Layer',
                'action': 'Добавление TCP заголовка',
                'details': f'Добавлены порты, sequence number, checksum',
                'data_size': len(data) + tcp_header_size,
                'protocol': 'TCP'
            })
            
            # Уровень 2 - Internet
            ip_header_size = 20
            journey.append({
                'layer': 2,
                'name': 'Internet Layer', 
                'action': 'Добавление IP заголовка',
                'details': f'Добавлены IP адреса источника и назначения',
                'data_size': len(data) + tcp_header_size + ip_header_size,
                'protocol': 'IPv4'
            })
            
            # Уровень 1 - Network Access
            ethernet_header_size = 14
            ethernet_trailer_size = 4
            journey.append({
                'layer': 1,
                'name': 'Network Access Layer',
                'action': 'Добавление Ethernet заголовка и трейлера',
                'details': f'Добавлены MAC адреса, CRC',
                'data_size': len(data) + tcp_header_size + ip_header_size + ethernet_header_size + ethernet_trailer_size,
                'protocol': 'Ethernet'
            })
            
            return journey
        
        # Пример отправки HTTP запроса
        example_journey = send_data('Web Browser', 'Web Server', 'GET /index.html HTTP/1.1')
        
        return {
            'scenario': 'HTTP запрос от браузера к веб-серверу',
            'journey': example_journey,
            'overhead_analysis': {
                'original_data': len('GET /index.html HTTP/1.1'),
                'tcp_overhead': 20,
                'ip_overhead': 20, 
                'ethernet_overhead': 18,
                'total_overhead': 58
            }
        }

# Демонстрация стека TCP/IP
def demonstrate_tcpip_stack():
    """Демонстрация стека протоколов TCP/IP"""
    
    stack = TCPIPStack()
    
    print("=== Стек протоколов TCP/IP ===\n")
    
    for layer_num in range(4, 0, -1):
        layer_info = stack.layers[layer_num]
        print(f"{layer_num}. {layer_info['name_ru']} ({layer_info['name']})")
        print(f"   Назначение: {layer_info['description']}")
        
        if 'protocols' in layer_info:
            print("   Основные протоколы:")
            for protocol, details in layer_info['protocols'].items():
                if isinstance(details, dict):
                    desc = details.get('description', '')
                    print(f"     • {protocol}: {desc}")
                else:
                    print(f"     • {protocol}")
        print()
    
    print("=== Сравнение с моделью OSI ===")
    comparison = stack.compare_with_osi()
    
    print("Основные различия:")
    for tcpip_layer, difference in comparison['differences'].items():
        print(f"• {tcpip_layer}: {difference}")
    
    print("\n=== Путешествие пакета ===")
    packet_sim = stack.packet_journey_simulation()
    print(f"Сценарий: {packet_sim['scenario']}\n")
    
    for step in packet_sim['journey']:
        print(f"Уровень {step['layer']} - {step['name']}:")
        print(f"  Действие: {step['action']}")
        print(f"  Детали: {step['details']}")
        print(f"  Размер данных: {step['data_size']} байт")
        if 'protocol' in step:
            print(f"  Протокол: {step['protocol']}")
        print()
    
    overhead = packet_sim['overhead_analysis']
    print("Анализ накладных расходов:")
    print(f"• Исходные данные: {overhead['original_data']} байт")
    print(f"• Накладные расходы TCP: {overhead['tcp_overhead']} байт")
    print(f"• Накладные расходы IP: {overhead['ip_overhead']} байт")
    print(f"• Накладные расходы Ethernet: {overhead['ethernet_overhead']} байт")
    print(f"• Общие накладные расходы: {overhead['total_overhead']} байт")
    efficiency = overhead['original_data'] / (overhead['original_data'] + overhead['total_overhead'])
    print(f"• Эффективность передачи: {efficiency:.1%}")

# Запуск демонстрации
demonstrate_tcpip_stack()
```

---

## Типы сетей

### Классификация по размеру и назначению

```python
class NetworkTypes:
    """Классификация типов компьютерных сетей"""
    
    def __init__(self):
        self.network_types = {
            'PAN': {
                'name': 'Personal Area Network',
                'name_ru': 'Персональная сеть',
                'range': '1-10 метров',
                'technologies': ['Bluetooth', 'USB', 'FireWire', 'Zigbee'],
                'examples': ['Соединение телефона с наушниками', 'Мышь и клавиатура', 'Синхронизация устройств'],
                'characteristics': {
                    'coverage': 'Вокруг одного человека',
                    'speed': '1 Mbps - 480 Mbps',
                    'topology': 'Звезда или шина'
                }
            },
            'LAN': {
                'name': 'Local Area Network',
                'name_ru': 'Локальная сеть',
                'range': '10 метров - 1 км',
                'technologies': ['Ethernet', 'Wi-Fi', 'Token Ring'],
                'examples': ['Офисная сеть', 'Домашняя сеть', 'Сеть учебного заведения'],
                'characteristics': {
                    'coverage': 'Здание или группа зданий',
                    'speed': '10 Mbps - 10 Gbps',
                    'topology': 'Звезда, шина, кольцо'
                }
            },
            'CAN': {
                'name': 'Campus Area Network',
                'name_ru': 'Кампусная сеть',
                'range': '1-5 км',
                'technologies': ['Fiber Optic', 'Ethernet'],
                'examples': ['Университетский кампус', 'Корпоративный комплекс', 'Больничный комплекс'],
                'characteristics': {
                    'coverage': 'Несколько зданий в одной местности',
                    'speed': '100 Mbps - 10 Gbps',
                    'topology': 'Иерархическая звезда'
                }
            },
            'MAN': {
                'name': 'Metropolitan Area Network',
                'name_ru': 'Городская сеть',
                'range': '5-50 км',
                'technologies': ['Fiber Optic', 'WiMAX', 'Cable'],
                'examples': ['Городская сеть кабельного TV', 'Сеть провайдера в городе'],
                'characteristics': {
                    'coverage': 'Город или большой район',
                    'speed': '10 Mbps - 1 Gbps',
                    'topology': 'Кольцо или звезда'
                }
            },
            'WAN': {
                'name': 'Wide Area Network',
                'name_ru': 'Глобальная сеть',
                'range': '50 км и более',
                'technologies': ['Satellite', 'Leased Lines', 'MPLS', 'Internet'],
                'examples': ['Интернет', 'Корпоративные сети между городами', 'Телефонные сети'],
                'characteristics': {
                    'coverage': 'Страны и континенты',
                    'speed': '56 Kbps - 100 Gbps',
                    'topology': 'Mesh (ячеистая)'
                }
            }
        }
        
        self.topologies = {
            'Bus': {
                'name_ru': 'Шина',
                'description': 'Все устройства подключены к одному кабелю',
                'advantages': ['Простота', 'Экономичность', 'Легкость расширения'],
                'disadvantages': ['Единая точка отказа', 'Коллизии', 'Сложность диагностики'],
                'used_in': ['Ранние Ethernet сети', 'Некоторые промышленные сети']
            },
            'Star': {
                'name_ru': 'Звезда',
                'description': 'Все устройства подключены к центральному узлу',
                'advantages': ['Централизованное управление', 'Легкость диагностики', 'Отказ одного узла не влияет на остальные'],
                'disadvantages': ['Центральный узел - единая точка отказа', 'Больше кабеля'],
                'used_in': ['Современные Ethernet сети', 'Wi-Fi сети']
            },
            'Ring': {
                'name_ru': 'Кольцо',
                'description': 'Устройства соединены в замкнутое кольцо',
                'advantages': ['Равный доступ к среде', 'Нет коллизий', 'Предсказуемая задержка'],
                'disadvantages': ['Отказ одного узла нарушает всю сеть', 'Сложность добавления узлов'],
                'used_in': ['Token Ring', 'FDDI', 'Некоторые промышленные сети']
            },
            'Mesh': {
                'name_ru': 'Ячеистая',
                'description': 'Каждое устройство соединено с несколькими другими',
                'advantages': ['Высокая надежность', 'Множественные пути', 'Отказоустойчивость'],
                'disadvantages': ['Высокая стоимость', 'Сложность управления', 'Много соединений'],
                'used_in': ['Интернет магистрали', 'Беспроводные mesh сети', 'Критически важные системы']
            },
            'Tree': {
                'name_ru': 'Дерево',
                'description': 'Иерархическая структура с корневым узлом',
                'advantages': ['Иерархическое управление', 'Масштабируемость', 'Централизованный контроль'],
                'disadvantages': ['Зависимость от корневых узлов', 'Возможные узкие места'],
                'used_in': ['Корпоративные сети', 'Кампусные сети', 'Провайдерские сети']
            }
        }
    
    def network_comparison(self):
        """Сравнение различных типов сетей"""
        
        comparison_table = []
        
        for net_type, info in self.network_types.items():
            comparison_table.append({
                'type': net_type,
                'name': info['name_ru'],
                'range': info['range'],
                'speed': info['characteristics']['speed'],
                'main_technology': info['technologies'][0],
                'typical_use': info['examples'][0]
            })
        
        return comparison_table
    
    def topology_analysis(self):
        """Анализ сетевых топологий"""
        
        analysis = {}
        
        for topology, info in self.topologies.items():
            analysis[topology] = {
                'name': info['name_ru'],
                'score': self._calculate_topology_score(info),
                'best_for': self._determine_best_use_case(info),
                'reliability': len(info['advantages']) - len(info['disadvantages'])
            }
        
        return analysis
    
    def _calculate_topology_score(self, topology_info):
        """Расчет оценки топологии (упрощенный)"""
        advantages = len(topology_info['advantages'])
        disadvantages = len(topology_info['disadvantages'])
        return max(1, advantages - disadvantages)
    
    def _determine_best_use_case(self, topology_info):
        """Определение лучшего применения для топологии"""
        if 'Wi-Fi' in str(topology_info['used_in']):
            return 'Малые и средние офисы'
        elif 'Интернет' in str(topology_info['used_in']):
            return 'Критически важные системы'
        elif 'промышленные' in str(topology_info['used_in']):
            return 'Промышленная автоматизация'
        else:
            return 'Общего назначения'
    
    def network_evolution_timeline(self):
        """Эволюция сетевых технологий"""
        
        timeline = {
            1960: {
                'event': 'ARPANET',
                'description': 'Первая сеть с коммутацией пакетов',
                'significance': 'Основа современного Интернета'
            },
            1973: {
                'event': 'Ethernet',
                'description': 'Роберт Меткалф изобретает Ethernet',
                'significance': 'Стандарт локальных сетей'
            },
            1983: {
                'event': 'TCP/IP становится стандартом',
                'description': 'ARPANET переходит на TCP/IP',
                'significance': 'Единый протокол для всех сетей'
            },
            1989: {
                'event': 'World Wide Web',
                'description': 'Тим Бернерс-Ли создает WWW',
                'significance': 'Революция в использовании Интернета'
            },
            1995: {
                'event': 'Коммерческий Интернет',
                'description': 'Снятие ограничений на коммерческое использование',
                'significance': 'Массовое распространение Интернета'
            },
            1997: {
                'event': 'Wi-Fi (802.11)',
                'description': 'Первый стандарт беспроводных сетей',
                'significance': 'Мобильность и повсеместный доступ'
            },
            2005: {
                'event': 'Broadband Internet',
                'description': 'Широкое распространение широкополосного доступа',
                'significance': 'Высокоскоростной Интернет для всех'
            },
            2012: {
                'event': 'IPv6 deployment',
                'description': 'Начало массового внедрения IPv6',
                'significance': 'Решение проблемы исчерпания IPv4 адресов'
            },
            2020: {
                'event': '5G Networks',
                'description': 'Развертывание сетей пятого поколения',
                'significance': 'Сверхбыстрая мобильная связь и IoT'
            }
        }
        
        return timeline

# Демонстрация типов сетей
def demonstrate_network_types():
    """Демонстрация различных типов сетей"""
    
    networks = NetworkTypes()
    
    print("=== Типы компьютерных сетей ===\n")
    
    comparison = networks.network_comparison()
    
    print("Сравнительная таблица сетей:")
    print(f"{'Тип':<6} {'Название':<15} {'Дальность':<15} {'Скорость':<20} {'Технология':<12}")
    print("-" * 80)
    
    for net in comparison:
        print(f"{net['type']:<6} {net['name']:<15} {net['range']:<15} {net['speed']:<20} {net['main_technology']:<12}")
    
    print("\n=== Детальная информация о типах сетей ===\n")
    
    for net_type, info in networks.network_types.items():
        print(f"{net_type} - {info['name_ru']} ({info['name']})")
        print(f"Дальность: {info['range']}")
        print(f"Технологии: {', '.join(info['technologies'])}")
        print(f"Примеры использования: {', '.join(info['examples'])}")
        print(f"Скорость: {info['characteristics']['speed']}")
        print(f"Покрытие: {info['characteristics']['coverage']}")
        print()
    
    print("=== Сетевые топологии ===\n")
    
    topology_analysis = networks.topology_analysis()
    
    for topology, analysis in topology_analysis.items():
        topo_info = networks.topologies[topology]
        print(f"{analysis['name']} ({topology})")
        print(f"Описание: {topo_info['description']}")
        print(f"Лучше всего для: {analysis['best_for']}")
        print(f"Оценка надежности: {analysis['reliability']}")
        print(f"Преимущества: {', '.join(topo_info['advantages'])}")
        print(f"Недостатки: {', '.join(topo_info['disadvantages'])}")
        print(f"Используется в: {', '.join(topo_info['used_in'])}")
        print()
    
    print("=== Эволюция сетевых технологий ===\n")
    
    timeline = networks.network_evolution_timeline()
    
    for year, event in timeline.items():
        print(f"{year}: {event['event']}")
        print(f"  {event['description']}")
        print(f"  Значение: {event['significance']}")
        print()

# Запуск демонстрации
demonstrate_network_types()
```

---

## Связанные разделы

- [[../fundamentals/architectural-styles|Архитектурные стили]] - Сетевые архитектуры
- [[../technical-skills/security|Безопасность]] - Сетевая безопасность
- [[linux-deployment|Linux развертывание]] - Настройка сетей в Linux
- [[docker-containerization|Docker]] - Сетевые возможности контейнеров
- [[../algorithms-data-structures/README|Алгоритмы]] - Алгоритмы маршрутизации 