







ИТОГОВАЯ РАБОТА




Название программы	«Инженер-тестировщик»
Группа обучения	«ИТ4-6»
Срок обучения	«05.08.2025 - 09.09.2025»
«‎Стрижков Олег Васильевич»
Номер/Название Кейса	«Задание 10. Создание фреймворка для автоматизированного тестирования пользовательских эндпоинтов»









Москва 2025 г.




Задание. Создание фреймворка для автоматизированного тестирования пользовательских эндпоинтов Описание задания: Вы разработаете простой тестовый фреймворк для тестирования API управления пользователями. Подготовка: 1. Создайте новый блокнот в Google Colab 2. Установите библиотеки requests, pytest и faker (для генерации тестовых данных) 3. Изучите документацию PetStore для методов работы с пользователями (/user) Задачи: 1. Создайте базовый класс для тестов с общими методами 2. Напишите функцию для генерации случайных тестовых данных пользователя 3. Реализуйте тесты для создания пользователя (POST /user) 4. Создайте тесты для входа и выхода пользователя (GET /user/login, /user/logout) 5. Напишите тесты для обновления и удаления пользователя 6. Добавьте функцию для создания отчета о выполнении тестов.
==============================================
Решение: (Ссылка Код на Google Disk - https://colab.research.google.com/drive/1UOPJ78GKV0wVqtywf_xmaZ20tm8Piipm?usp=sharing ) 
Код автотеста для применения на сайте Colab.research.google.com

# @title
# Установка необходимых библиотек
!pip install requests faker


import requests
import json
import random
from faker import Faker
from typing import Dict, Any
import datetime
import sys

class UserAPITestBase:
    """Базовый класс для тестирования API пользователей"""

    BASE_URL = "https://petstore.swagger.io/v2"

    def __init__(self):
        self.fake = Faker()
        self.session = requests.Session()
        self.test_results = []

    def generate_user_data(self) -> Dict[str, Any]:
        """Генерация случайных тестовых данных пользователя"""
        return {
            "id": random.randint(1000, 9999),
            "username": self.fake.user_name(),
            "firstName": self.fake.first_name(),
            "lastName": self.fake.last_name(),
            "email": self.fake.email(),
            "password": self.fake.password(length=12),
            "phone": self.fake.phone_number(),
            "userStatus": random.randint(0, 1)
        }

    def create_user(self, user_data: Dict[str, Any]) -> requests.Response:
        """Создание пользователя"""
        url = f"{self.BASE_URL}/user"
        headers = {'Content-Type': 'application/json'}
        return self.session.post(url, data=json.dumps(user_data), headers=headers)

    def login_user(self, username: str, password: str) -> requests.Response:
        """Вход пользователя"""
        url = f"{self.BASE_URL}/user/login?username={username}&password={password}"
        return self.session.get(url)

    def logout_user(self) -> requests.Response:
        """Выход пользователя"""
        url = f"{self.BASE_URL}/user/logout"
        return self.session.get(url)

    def get_user(self, username: str) -> requests.Response:
        """Получение информации о пользователе"""
        url = f"{self.BASE_URL}/user/{username}"
        return self.session.get(url)

    def update_user(self, username: str, user_data: Dict[str, Any]) -> requests.Response:
        """Обновление пользователя"""
        url = f"{self.BASE_URL}/user/{username}"
        headers = {'Content-Type': 'application/json'}
        return self.session.put(url, data=json.dumps(user_data), headers=headers)

    def delete_user(self, username: str) -> requests.Response:
        """Удаление пользователя"""
        url = f"{self.BASE_URL}/user/{username}"
        return self.session.delete(url)

    def add_test_result(self, test_name: str, status: str, response: requests.Response,
                       expected_status: int = None, notes: str = ""):
        """Добавление результата теста"""
        result = {
            "test_name": test_name,
            "status": status,
            "timestamp": datetime.datetime.now().isoformat(),
            "response_status": response.status_code,
            "expected_status": expected_status,
            "response_body": response.json() if response.content else {},
            "notes": notes
        }
        self.test_results.append(result)

    def generate_report(self):
        """Генерация отчета о тестировании"""
        total_tests = len(self.test_results)
        passed_tests = sum(1 for result in self.test_results if result["status"] == "PASSED")
        failed_tests = total_tests - passed_tests

        report = {
            "report_date": datetime.datetime.now().isoformat(),
            "total_tests": total_tests,
            "passed_tests": passed_tests,
            "failed_tests": failed_tests,
            "success_rate": (passed_tests / total_tests * 100) if total_tests > 0 else 0,
            "test_results": self.test_results
        }

        # Сохранение отчета в файл
        with open("test_report.json", "w") as f:
            json.dump(report, f, indent=2)

        # Вывод отчета в консоль
        print("=" * 60)
        print("ОТЧЕТ О ТЕСТИРОВАНИИ API ПОЛЬЗОВАТЕЛЕЙ")
        print("=" * 60)
        print(f"Дата отчета: {datetime.datetime.now()}")
        print(f"Всего тестов: {total_tests}")
        print(f"Пройдено: {passed_tests}")
        print(f"Не пройдено: {failed_tests}")
        print(f"Успешность: {report['success_rate']:.2f}%")
        print("=" * 60)

        for result in self.test_results:
            status_icon = "✅" if result["status"] == "PASSED" else "❌"
            print(f"{status_icon} {result['test_name']}: {result['status']}")
            if result["status"] == "FAILED":
                print(f"   Ожидалось: {result['expected_status']}, Получено: {result['response_status']}")
                print(f"   Примечание: {result['notes']}")

        return report

class TestUserCreation(UserAPITestBase):
    """Тесты для создания пользователей"""

    def test_create_user_success(self):
        """Тест успешного создания пользователя"""
        user_data = self.generate_user_data()
        response = self.create_user(user_data)

        if response.status_code == 200:
            self.add_test_result("test_create_user_success", "PASSED", response, 200)
            return user_data
        else:
            self.add_test_result("test_create_user_success", "FAILED", response, 200,
                               "Не удалось создать пользователя")
            return None

    def test_create_user_invalid_data(self):
        """Тест создания пользователя с невалидными данными"""
        invalid_data = {"invalid": "data"}
        url = f"{self.BASE_URL}/user"
        response = self.session.post(url, data=json.dumps(invalid_data),
                                   headers={'Content-Type': 'application/json'})

        if response.status_code != 200 or "error" in str(response.json()).lower():
            self.add_test_result("test_create_user_invalid_data", "PASSED", response,
                               "any error", "Ожидалась ошибка при невалидных данных")
        else:
            self.add_test_result("test_create_user_invalid_data", "FAILED", response,
                               "error expected", "Не возвращена ошибка на невалидные данные")

class TestUserLoginLogout(UserAPITestBase):
    """Тесты для входа и выхода пользователя"""

    def test_login_logout_success(self):
        """Тест успешного входа и выхода пользователя"""
        user_data = self.generate_user_data()
        create_response = self.create_user(user_data)

        if create_response.status_code != 200:
            self.add_test_result("test_login_logout_success", "FAILED", create_response, 200,
                               "Не удалось создать пользователя для теста входа")
            return

        login_response = self.login_user(user_data["username"], user_data["password"])

        if login_response.status_code == 200 and "logged in" in login_response.text.lower():
            self.add_test_result("test_login_success", "PASSED", login_response, 200)
        else:
            self.add_test_result("test_login_success", "FAILED", login_response, 200,
                               "Не удалось войти пользователя")
            return

        logout_response = self.logout_user()

        if logout_response.status_code == 200 and "ok" in logout_response.text.lower():
            self.add_test_result("test_logout_success", "PASSED", logout_response, 200)
        else:
            self.add_test_result("test_logout_success", "FAILED", logout_response, 200,
                               "Не удалось выйти пользователя")

    def test_login_invalid_credentials(self):
        """Тест входа с неверными учетными данными"""
        response = self.login_user("nonexistent_user", "wrong_password")

        if response.status_code == 200 and "invalid" in response.text.lower():
            self.add_test_result("test_login_invalid_credentials", "PASSED", response,
                               "any error", "Ожидалась ошибка при неверных учетных данных")
        else:
            self.add_test_result("test_login_invalid_credentials", "FAILED", response,
                               "error expected", "Не возвращена ошибка при неверных учетных данных")

class TestUserUpdateDelete(UserAPITestBase):
    """Тесты для обновления и удаления пользователя"""

    def test_update_user_success(self):
        """Тест успешного обновления пользователя"""
        user_data = self.generate_user_data()
        create_response = self.create_user(user_data)

        if create_response.status_code != 200:
            self.add_test_result("test_update_user_success", "FAILED", create_response, 200,
                               "Не удалось создать пользователя для теста обновления")
            return

        updated_data = user_data.copy()
        updated_data["firstName"] = "UpdatedFirstName"
        updated_data["email"] = "updated@example.com"

        update_response = self.update_user(user_data["username"], updated_data)

        if update_response.status_code == 200:
            self.add_test_result("test_update_user_success", "PASSED", update_response, 200)

            get_response = self.get_user(user_data["username"])
            if (get_response.status_code == 200 and
                get_response.json()["firstName"] == "UpdatedFirstName"):
                self.add_test_result("test_verify_update", "PASSED", get_response, 200)
            else:
                self.add_test_result("test_verify_update", "FAILED", get_response, 200,
                                   "Данные пользователя не обновились")
        else:
            self.add_test_result("test_update_user_success", "FAILED", update_response, 200,
                               "Не удалось обновить пользователя")

    def test_delete_user_success(self):
        """Тест успешного удаления пользователя"""
        user_data = self.generate_user_data()
        create_response = self.create_user(user_data)

        if create_response.status_code != 200:
            self.add_test_result("test_delete_user_success", "FAILED", create_response, 200,
                               "Не удалось создать пользователя для теста удаления")
            return

        delete_response = self.delete_user(user_data["username"])

        if delete_response.status_code == 200:
            self.add_test_result("test_delete_user_success", "PASSED", delete_response, 200)

            get_response = self.get_user(user_data["username"])
            if get_response.status_code == 404:
                self.add_test_result("test_verify_deletion", "PASSED", get_response, 404)
            else:
                self.add_test_result("test_verify_deletion", "FAILED", get_response, 404,
                                   "Пользователь не был удален")
        else:
            self.add_test_result("test_delete_user_success", "FAILED", delete_response, 200,
                               "Не удалось удалить пользователя")

def run_all_tests():
    """Запуск всех тестов и генерация отчета"""

    print("Запуск тестов API пользователей...")
    print("=" * 50)

    creation_tests = TestUserCreation()
    login_tests = TestUserLoginLogout()
    update_tests = TestUserUpdateDelete()

    print("Тестирование создания пользователей...")
    user_data = creation_tests.test_create_user_success()
    creation_tests.test_create_user_invalid_data()

    print("Тестирование входа/выхода пользователей...")
    login_tests.test_login_logout_success()
    login_tests.test_login_invalid_credentials()

    print("Тестирование обновления и удаления пользователей...")
    update_tests.test_update_user_success()
    update_tests.test_delete_user_success()

    all_results = (creation_tests.test_results +
                  login_tests.test_results +
                  update_tests.test_results)

    final_report = UserAPITestBase()
    final_report.test_results = all_results
    report = final_report.generate_report()

    print("\nТестирование завершено!")
    return report

def test_single_user_flow():
    """Тест полного цикла работы с пользователем"""
    print("Тестирование полного цикла пользователя...")

    base = UserAPITestBase()
    user_data = base.generate_user_data()

    steps = [
        ("Создание пользователя", lambda: base.create_user(user_data), 200),
        ("Вход пользователя", lambda: base.login_user(user_data["username"], user_data["password"]), 200),
        ("Получение информации", lambda: base.get_user(user_data["username"]), 200),
        ("Выход пользователя", lambda: base.logout_user(), 200),
        ("Удаление пользователя", lambda: base.delete_user(user_data["username"]), 200)
    ]

    for step_name, step_func, expected_status in steps:
        print(f"{steps.index((step_name, step_func, expected_status)) + 1}. {step_name}...")
        response = step_func()
        status = "PASSED" if response.status_code == expected_status else "FAILED"
        base.add_test_result(f"full_flow_{step_name.lower().replace(' ', '_')}",
                           status, response, expected_status)

    base.generate_report()

if __name__ == "__main__":
    print("Фреймворк для тестирования API пользователей PetStore")
    print("Выберите вариант запуска:")
    print("1 - Запуск всех тестов")
    print("2 - Тест полного цикла пользователя")
    print("3 - Генерация тестовых данных")

    choice = input("Введите номер (1-3): ").strip()

    if choice == "1":
        run_all_tests()
    elif choice == "2":
        test_single_user_flow()
    elif choice == "3":
        base = UserAPITestBase()
        test_user = base.generate_user_data()
        print("Пример тестовых данных пользователя:")
        print(json.dumps(test_user, indent=2))
    else:
        print("Неверный выбор. Запуск всех тестов...")
        run_all_tests()

Скриншот (код работает на сайте Colab.research.google.com): 


Код , для запуска в vs code (Ссылка на весь проект на GitHub - https://github.com/olegsamara-63/itog_work_Oleg_S_make_to_framework_for_RT.git , SSH - git@github.com:olegsamara-63/itog_work_Oleg_S_make_to_framework_for_RT.git , 
GitLens  - gh repo clone olegsamara-63/itog_work_Oleg_S_make_to_framework_for_RT)

Код Python:

import requests
import json
import random
from faker import Faker
from typing import Dict, Any
import datetime
import sys

class UserAPITestBase:
    """Базовый класс для тестирования API пользователей"""
    
    BASE_URL = "https://petstore.swagger.io/v2"
    
    def __init__(self):
        self.fake = Faker()
        self.session = requests.Session()
        self.test_results = []
    
    def generate_user_data(self) -> Dict[str, Any]:
        """Генерация случайных тестовых данных пользователя"""
        return {
            "id": random.randint(1000, 9999),
            "username": self.fake.user_name(),
            "firstName": self.fake.first_name(),
            "lastName": self.fake.last_name(),
            "email": self.fake.email(),
            "password": self.fake.password(length=12),
            "phone": self.fake.phone_number(),
            "userStatus": random.randint(0, 1)
        }
    
    def create_user(self, user_data: Dict[str, Any]) -> requests.Response:
        """Создание пользователя"""
        url = f"{self.BASE_URL}/user"
        headers = {'Content-Type': 'application/json'}
        return self.session.post(url, data=json.dumps(user_data), headers=headers)
    
    def login_user(self, username: str, password: str) -> requests.Response:
        """Вход пользователя"""
        url = f"{self.BASE_URL}/user/login?username={username}&password={password}"
        return self.session.get(url)
    
    def logout_user(self) -> requests.Response:
        """Выход пользователя"""
        url = f"{self.BASE_URL}/user/logout"
        return self.session.get(url)
    
    def get_user(self, username: str) -> requests.Response:
        """Получение информации о пользователе"""
        url = f"{self.BASE_URL}/user/{username}"
        return self.session.get(url)
    
    def update_user(self, username: str, user_data: Dict[str, Any]) -> requests.Response:
        """Обновление пользователя"""
        url = f"{self.BASE_URL}/user/{username}"
        headers = {'Content-Type': 'application/json'}
        return self.session.put(url, data=json.dumps(user_data), headers=headers)
    
    def delete_user(self, username: str) -> requests.Response:
        """Удаление пользователя"""
        url = f"{self.BASE_URL}/user/{username}"
        return self.session.delete(url)
    
    def add_test_result(self, test_name: str, status: str, response: requests.Response, 
                       expected_status: int = None, notes: str = ""):
        """Добавление результата теста"""
        result = {
            "test_name": test_name,
            "status": status,
            "timestamp": datetime.datetime.now().isoformat(),
            "response_status": response.status_code,
            "expected_status": expected_status,
            "response_body": response.json() if response.content else {},
            "notes": notes
        }
        self.test_results.append(result)
    
    def generate_report(self):
        """Генерация отчета о тестировании"""
        total_tests = len(self.test_results)
        passed_tests = sum(1 for result in self.test_results if result["status"] == "PASSED")
        failed_tests = total_tests - passed_tests
        
        report = {
            "report_date": datetime.datetime.now().isoformat(),
            "total_tests": total_tests,
            "passed_tests": passed_tests,
            "failed_tests": failed_tests,
            "success_rate": (passed_tests / total_tests * 100) if total_tests > 0 else 0,
            "test_results": self.test_results
        }
        
        # Сохранение отчета в файл
        with open("test_report.json", "w") as f:
            json.dump(report, f, indent=2)
        
        # Вывод отчета в консоль
        print("=" * 60)
        print("ОТЧЕТ О ТЕСТИРОВАНИИ API ПОЛЬЗОВАТЕЛЕЙ")
        print("=" * 60)
        print(f"Дата отчета: {datetime.datetime.now()}")
        print(f"Всего тестов: {total_tests}")
        print(f"Пройдено: {passed_tests}")
        print(f"Не пройдено: {failed_tests}")
        print(f"Успешность: {report['success_rate']:.2f}%")
        print("=" * 60)
        
        for result in self.test_results:
            status_icon = "✅" if result["status"] == "PASSED" else "❌"
            print(f"{status_icon} {result['test_name']}: {result['status']}")
            if result["status"] == "FAILED":
                print(f"   Ожидалось: {result['expected_status']}, Получено: {result['response_status']}")
                print(f"   Примечание: {result['notes']}")
        
        return report

class TestUserCreation(UserAPITestBase):
    """Тесты для создания пользователей"""
    
    def test_create_user_success(self):
        """Тест успешного создания пользователя"""
        user_data = self.generate_user_data()
        response = self.create_user(user_data)
        
        if response.status_code == 200:
            self.add_test_result("test_create_user_success", "PASSED", response, 200)
            return user_data
        else:
            self.add_test_result("test_create_user_success", "FAILED", response, 200, 
                               "Не удалось создать пользователя")
            return None
    
    def test_create_user_invalid_data(self):
        """Тест создания пользователя с невалидными данными"""
        invalid_data = {"invalid": "data"}
        url = f"{self.BASE_URL}/user"
        response = self.session.post(url, data=json.dumps(invalid_data), 
                                   headers={'Content-Type': 'application/json'})
        
        if response.status_code != 200 or "error" in str(response.json()).lower():
            self.add_test_result("test_create_user_invalid_data", "PASSED", response, 
                               "any error", "Ожидалась ошибка при невалидных данных")
        else:
            self.add_test_result("test_create_user_invalid_data", "FAILED", response, 
                               "error expected", "Не возвращена ошибка на невалидные данные")

class TestUserLoginLogout(UserAPITestBase):
    """Тесты для входа и выхода пользователя"""
    
    def test_login_logout_success(self):
        """Тест успешного входа и выхода пользователя"""
        user_data = self.generate_user_data()
        create_response = self.create_user(user_data)
        
        if create_response.status_code != 200:
            self.add_test_result("test_login_logout_success", "FAILED", create_response, 200,
                               "Не удалось создать пользователя для теста входа")
            return
        
        login_response = self.login_user(user_data["username"], user_data["password"])
        
        if login_response.status_code == 200 and "logged in" in login_response.text.lower():
            self.add_test_result("test_login_success", "PASSED", login_response, 200)
        else:
            self.add_test_result("test_login_success", "FAILED", login_response, 200,
                               "Не удалось войти пользователя")
            return
        
        logout_response = self.logout_user()
        
        if logout_response.status_code == 200 and "ok" in logout_response.text.lower():
            self.add_test_result("test_logout_success", "PASSED", logout_response, 200)
        else:
            self.add_test_result("test_logout_success", "FAILED", logout_response, 200,
                               "Не удалось выйти пользователя")
    
    def test_login_invalid_credentials(self):
        """Тест входа с неверными учетными данными"""
        response = self.login_user("nonexistent_user", "wrong_password")
        
        if response.status_code == 200 and "invalid" in response.text.lower():
            self.add_test_result("test_login_invalid_credentials", "PASSED", response, 
                               "any error", "Ожидалась ошибка при неверных учетных данных")
        else:
            self.add_test_result("test_login_invalid_credentials", "FAILED", response,
                               "error expected", "Не возвращена ошибка при неверных учетных данных")

class TestUserUpdateDelete(UserAPITestBase):
    """Тесты для обновления и удаления пользователя"""
    
    def test_update_user_success(self):
        """Тест успешного обновления пользователя"""
        user_data = self.generate_user_data()
        create_response = self.create_user(user_data)
        
        if create_response.status_code != 200:
            self.add_test_result("test_update_user_success", "FAILED", create_response, 200,
                               "Не удалось создать пользователя для теста обновления")
            return
        
        updated_data = user_data.copy()
        updated_data["firstName"] = "UpdatedFirstName"
        updated_data["email"] = "updated@example.com"
        
        update_response = self.update_user(user_data["username"], updated_data)
        
        if update_response.status_code == 200:
            self.add_test_result("test_update_user_success", "PASSED", update_response, 200)
            
            get_response = self.get_user(user_data["username"])
            if (get_response.status_code == 200 and 
                get_response.json()["firstName"] == "UpdatedFirstName"):
                self.add_test_result("test_verify_update", "PASSED", get_response, 200)
            else:
                self.add_test_result("test_verify_update", "FAILED", get_response, 200,
                                   "Данные пользователя не обновились")
        else:
            self.add_test_result("test_update_user_success", "FAILED", update_response, 200,
                               "Не удалось обновить пользователя")
    
    def test_delete_user_success(self):
        """Тест успешного удаления пользователя"""
        user_data = self.generate_user_data()
        create_response = self.create_user(user_data)
        
        if create_response.status_code != 200:
            self.add_test_result("test_delete_user_success", "FAILED", create_response, 200,
                               "Не удалось создать пользователя для теста удаления")
            return
        
        delete_response = self.delete_user(user_data["username"])
        
        if delete_response.status_code == 200:
            self.add_test_result("test_delete_user_success", "PASSED", delete_response, 200)
            
            get_response = self.get_user(user_data["username"])
            if get_response.status_code == 404:
                self.add_test_result("test_verify_deletion", "PASSED", get_response, 404)
            else:
                self.add_test_result("test_verify_deletion", "FAILED", get_response, 404,
                                   "Пользователь не был удален")
        else:
            self.add_test_result("test_delete_user_success", "FAILED", delete_response, 200,
                               "Не удалось удалить пользователя")

def run_all_tests():
    """Запуск всех тестов и генерация отчета"""
    
    print("Запуск тестов API пользователей...")
    print("=" * 50)
    
    creation_tests = TestUserCreation()
    login_tests = TestUserLoginLogout()
    update_tests = TestUserUpdateDelete()
    
    print("Тестирование создания пользователей...")
    user_data = creation_tests.test_create_user_success()
    creation_tests.test_create_user_invalid_data()
    
    print("Тестирование входа/выхода пользователей...")
    login_tests.test_login_logout_success()
    login_tests.test_login_invalid_credentials()
    
    print("Тестирование обновления и удаления пользователей...")
    update_tests.test_update_user_success()
    update_tests.test_delete_user_success()
    
    all_results = (creation_tests.test_results + 
                  login_tests.test_results + 
                  update_tests.test_results)
    
    final_report = UserAPITestBase()
    final_report.test_results = all_results
    report = final_report.generate_report()
    
    print("\nТестирование завершено!")
    return report

def test_single_user_flow():
    """Тест полного цикла работы с пользователем"""
    print("Тестирование полного цикла пользователя...")
    
    base = UserAPITestBase()
    user_data = base.generate_user_data()
    
    steps = [
        ("Создание пользователя", lambda: base.create_user(user_data), 200),
        ("Вход пользователя", lambda: base.login_user(user_data["username"], user_data["password"]), 200),
        ("Получение информации", lambda: base.get_user(user_data["username"]), 200),
        ("Выход пользователя", lambda: base.logout_user(), 200),
        ("Удаление пользователя", lambda: base.delete_user(user_data["username"]), 200)
    ]
    
    for step_name, step_func, expected_status in steps:
        print(f"{steps.index((step_name, step_func, expected_status)) + 1}. {step_name}...")
        response = step_func()
        status = "PASSED" if response.status_code == expected_status else "FAILED"
        base.add_test_result(f"full_flow_{step_name.lower().replace(' ', '_')}", 
                           status, response, expected_status)
    
    base.generate_report()

if __name__ == "__main__":
    print("Фреймворк для тестирования API пользователей PetStore")
    print("Выберите вариант запуска:")
    print("1 - Запуск всех тестов")
    print("2 - Тест полного цикла пользователя")
    print("3 - Генерация тестовых данных")
    
    choice = input("Введите номер (1-3): ").strip()
    
    if choice == "1":
        run_all_tests()
    elif choice == "2":
        test_single_user_flow()
    elif choice == "3":
        base = UserAPITestBase()
        test_user = base.generate_user_data()
        print("Пример тестовых данных пользователя:")
        print(json.dumps(test_user, indent=2))
    else:
        print("Неверный выбор. Запуск всех тестов...")
        run_all_tests()


        
Скриншот (код работает в IDLE VS Code):



Файл pyvenv.cfg
home = /usr/bin
include-system-site-packages = false
version = 3.12.7
executable = /usr/bin/python3.12
command = /usr/bin/python3 -m venv /home/oleg/itog_work_Oleg_S_make_to_framework_for_RT/venv

Файл requirements.txt :
requests==2.31.0
faker==19.6.2
pytest
pytest-html
faker

Файл test_report.json :
{
  "report_date": "2025-08-31T05:14:56.086123",
  "total_tests": 9,
  "passed_tests": 6,
  "failed_tests": 3,
  "success_rate": 66.6666666666667,
  "test_results": [
    {
      "test_name": "test_create_user_success",
      "status": "PASSED",
      "timestamp": "2025-08-31T05:14:52.124130",
      "response_status": 200,
      "expected_status": 200,
      "response_body": {
        "code": 200,
        "type": "unknown",
        "message": "7686"
      },
      "notes": ""
    },
    {
      "test_name": "test_create_user_invalid_data",
      "status": "FAILED",
      "timestamp": "2025-08-31T05:14:52.394523",
      "response_status": 200,
      "expected_status": "error expected",
      "response_body": {
        "code": 200,
        "type": "unknown",
        "message": "0"
      },
      "notes": "Не возвращена ошибка на невалидные данные"
    },
    {
      "test_name": "test_login_success",
      "status": "PASSED",
      "timestamp": "2025-08-31T05:14:53.622168",
      "response_status": 200,
      "expected_status": 200,
      "response_body": {
        "code": 200,
        "type": "unknown",
        "message": "logged in user session:1756602893536"
      },
      "notes": ""
    },
    {
      "test_name": "test_logout_success",
      "status": "PASSED",
      "timestamp": "2025-08-31T05:14:53.833137",
      "response_status": 200,
      "expected_status": 200,
      "response_body": {
        "code": 200,
        "type": "unknown",
        "message": "ok"
      },
      "notes": ""
    },
    {
      "test_name": "test_login_invalid_credentials",
      "status": "FAILED",
      "timestamp": "2025-08-31T05:14:54.000846",
      "response_status": 200,
      "expected_status": "error expected",
      "response_body": {
        "code": 200,
        "type": "unknown",
        "message": "logged in user session:1756602893912"
      },
      "notes": "Не возвращена ошибка при неверных учетных данных"
    },
    {
      "test_name": "test_update_user_success",
      "status": "PASSED",
      "timestamp": "2025-08-31T05:14:55.260845",
      "response_status": 200,
      "expected_status": 200,
      "response_body": {
        "code": 200,
        "type": "unknown",
        "message": "9519"
      },
      "notes": ""
    },
    {
      "test_name": "test_verify_update",
      "status": "FAILED",
      "timestamp": "2025-08-31T05:14:55.478034",
      "response_status": 200,
      "expected_status": 200,
      "response_body": {
        "id": 9519,
        "username": "yvonnepeters",
        "firstName": "Ernest",
        "lastName": "Hansen",
        "email": "hatfieldstacey@example.net",
        "password": "6zz0UTp4m(HN",
        "phone": "001-936-896-6272x8745",
        "userStatus": 1
      },
      "notes": "Данные пользователя не обновились"
    },
    {
      "test_name": "test_delete_user_success",
      "status": "PASSED",
      "timestamp": "2025-08-31T05:14:55.877286",
      "response_status": 200,
      "expected_status": 200,
      "response_body": {
        "code": 200,
        "type": "unknown",
        "message": "daniellereilly"
      },
      "notes": ""
    },
    {
      "test_name": "test_verify_deletion",
      "status": "PASSED",
      "timestamp": "2025-08-31T05:14:56.082267",
      "response_status": 404,
      "expected_status": 404,
      "response_body": {
        "code": 1,
        "type": "error",
        "message": "User not found"
      },
      "notes": ""
    }
  ]
}















Скриншот json отчёта:











