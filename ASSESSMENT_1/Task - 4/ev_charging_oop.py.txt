class User:

    def __init__(self, user_id, name, user_type, first_time, eco_pass):
        self.__user_id = user_id
        self.__name = name
        self.__user_type = user_type
        self.__first_time = first_time
        self.__eco_pass = eco_pass

    # Getters and setters

    def get_user_id(self):
        return self.__user_id

    def set_user_id(self, user_id):
        if user_id:
            self.__user_id = user_id

    def get_name(self):
        return self.__name

    def set_name(self, name):
        if name:
            self.__name = name

    def get_user_type(self):
        return self.__user_type

    def set_user_type(self, user_type):
        if user_type in ["STUDENT", "STAFF", "NONE"]:
            self.__user_type = user_type

    def is_first_time(self):
        return self.__first_time

    def set_first_time(self, value):
        self.__first_time = value

    def has_eco_pass(self):
        return self.__eco_pass

    def set_eco_pass(self, value):
        self.__eco_pass = value


    # Polymorphic method

    def calculate_fee(self, session):
        return session.calculate_base_fee()


class MemberUser(User):
    def __init__(
        self, user_id, name, user_type, first_time,
        eco_pass, membership_type, discount_rate
    ):
        super().__init__(
            user_id, name, user_type, first_time, eco_pass
        )
        self.__membership_type = membership_type
        self.__discount_rate = discount_rate

    def get_membership_type(self):
        return self.__membership_type

    def set_membership_type(self, membership_type):
        if membership_type in ["STUDENT", "STAFF"]:
            self.__membership_type = membership_type

    def get_discount_rate(self):
        return self.__discount_rate

    def set_discount_rate(self, rate):
        if 0 <= rate <= 1:
            self.__discount_rate = rate


    # Method overriding - polymorphism

    def calculate_fee(self, session):
        gross_fee = session.calculate_base_fee()

        if self.is_first_time():
            discount = gross_fee
        elif self.__membership_type == "STAFF":
            discount = gross_fee * 0.50
        elif (
            self.__membership_type == "STUDENT"
            and session.get_charger_type() == "AC"
        ):
            discount = gross_fee * 0.25
        else:
            discount = 0

        return max(gross_fee - discount, 0)


class EVCharger:
    def __init__(self, charger_id, charger_type, hourly_rate, is_peak):
        self.__charger_id = charger_id
        self.__charger_type = charger_type
        self.__hourly_rate = hourly_rate
        self.__is_peak = is_peak

    def get_charger_id(self):
        return self.__charger_id

    def set_charger_id(self, charger_id):
        if charger_id:
            self.__charger_id = charger_id

    def get_charger_type(self):
        return self.__charger_type

    def set_charger_type(self, charger_type):
        if charger_type in ["AC", "DC"]:
            self.__charger_type = charger_type

    def get_hourly_rate(self):
        return self.__hourly_rate

    def set_hourly_rate(self, rate):
        if rate >= 0:
            self.__hourly_rate = rate

    def is_peak(self):
        return self.__is_peak

    def set_peak(self, value):
        self.__is_peak = value

    def calculate_base_fee(self, hours):
        if self.__charger_type == "AC":
            if hours <= 2:
                rate = 4
            elif hours <= 4:
                rate = 6
            elif hours <= 6:
                rate = 8
            else:
                rate = 12

            fee = rate * hours
            return min(fee, 80) if hours > 6 else fee

        if hours <= 2:
            rate = 10
        elif hours <= 4:
            rate = 15
        elif hours <= 6:
            rate = 20
        else:
            rate = 30

        fee = rate * hours
        return min(fee, 150) if hours > 6 else fee


class ChargingSession:
    def __init__(
        self, session_id, user, charger,
        hours, idle_parking, lost_rfid
    ):
        self.__session_id = session_id
        self.__user = user
        self.__charger = charger
        self.__hours = hours
        self.__idle_parking = idle_parking
        self.__lost_rfid = lost_rfid
        self.__total_fee = 0

    def get_session_id(self):
        return self.__session_id

    def set_session_id(self, session_id):
        if session_id:
            self.__session_id = session_id

    def get_user(self):
        return self.__user

    def set_user(self, user):
        self.__user = user

    def get_charger(self):
        return self.__charger

    def set_charger(self, charger):
        self.__charger = charger

    def get_charger_type(self):
        return self.__charger.get_charger_type()

    def get_total_fee(self):
        return self.__total_fee

    def calculate_base_fee(self):
        return self.__charger.calculate_base_fee(self.__hours)

    def apply_waiver(self, amount):
        return max(amount, 0)

    def add_surcharges(self):
        surcharge = 0

        if self.__charger.is_peak():
            surcharge += 5

        if self.__idle_parking:
            surcharge += 15

        if self.__lost_rfid:
            surcharge += 30

        return surcharge

    def generate_bill(self):
        gross_fee = self.calculate_base_fee()
        discounted_fee = self.__user.calculate_fee(self)

        member_discount = gross_fee - discounted_fee

        eco_pass_discount = 2 if self.__user.has_eco_pass() else 0

        surcharge = self.add_surcharges()

        self.__total_fee = max(
            discounted_fee - eco_pass_discount + surcharge, 0
        )

        return (
            gross_fee,
            member_discount,
            eco_pass_discount,
            surcharge,
            self.__total_fee
        )


def main():
    process_another = "YES"
    session_id = 1

    while process_another == "YES":

        print("\n--- Taylor's Campus EV Charging System ---")

        user_id = input("Enter User ID: ")
        name = input("Enter Name: ")
        vehicle_number = input("Enter Vehicle Number: ")

        user_type = input(
            "Enter Member Type (STUDENT/STAFF/NONE): "
        ).upper()

        charger_type = input(
            "Enter Charger Type (AC/DC): "
        ).upper()

        hours = int(input("Enter Hours Charged: "))

        first_time = input(
            "Is this a first-time user? (YES/NO): "
        ).upper() == "YES"

        eco_pass = input(
            "Has Green Eco-Pass? (YES/NO): "
        ).upper() == "YES"

        peak_hour = input(
            "Is it peak hour? (YES/NO): "
        ).upper() == "YES"

        idle_parking = input(
            "Is there idle parking? (YES/NO): "
        ).upper() == "YES"

        lost_rfid = input(
            "Lost RFID card? (YES/NO): "
        ).upper() == "YES"

        # Create user object
        if user_type == "STUDENT":
            user = MemberUser(
                user_id, name, user_type, first_time,
                eco_pass, "STUDENT", 0.25
            )
        elif user_type == "STAFF":
            user = MemberUser(
                user_id, name, user_type, first_time,
                eco_pass, "STAFF", 0.50
            )
        else:
            user = User(
                user_id, name, user_type,
                first_time, eco_pass
            )


        # Create charger and charging session objects

        charger = EVCharger(
            session_id, charger_type, 0, peak_hour
        )

        session = ChargingSession(
            session_id,
            user,
            charger,
            hours,
            idle_parking,
            lost_rfid
        )


        # Generate bill

        (
            gross_fee,
            member_discount,
            eco_pass_discount,
            surcharge,
            net_payable
        ) = session.generate_bill()

        print("\n----------------------------------------")
        print("Taylor's Campus EV Charging Bill")
        print("----------------------------------------")
        print("User ID:", user.get_user_id())
        print("Name:", user.get_name())
        print("Vehicle Number:", vehicle_number)
        print("Member Type:", user.get_user_type())
        print("Charger Type:", charger.get_charger_type())
        print("Charging Hours:", hours)
        print(f"Gross Charging Fee: RM {gross_fee:.2f}")
        print(f"Member Discount: RM {member_discount:.2f}")
        print(f"Eco-Pass Discount: RM {eco_pass_discount:.2f}")
        print(f"Total Surcharges: RM {surcharge:.2f}")
        print(f"Net Payable: RM {net_payable:.2f}")
        print("----------------------------------------")

        process_another = input(
            "Process another vehicle? (YES/NO): "
        ).upper()

        session_id += 1

    print("Program ended.")


if __name__ == "__main__":
    main()
