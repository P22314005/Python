# Python
# 示例：Flask-JWT认证路由
from flask import Flask, request, jsonify
from flask_jwt_extended import create_access_token, jwt_required, get_jwt_identity

app = Flask(__name__)
app.config["JWT_SECRET_KEY"] = "your-secret-key"

# 登录接口
@app.route("/api/login", methods=["POST"])
def login():
    username = request.json.get("username", None)
    password = request.json.get("password", None)
    # 验证用户名密码（实际需对接数据库）
    if username == "admin" and password == "admin123":
        access_token = create_access_token(identity=username)
        return jsonify(access_token=access_token), 200
    return jsonify({"msg": "登录失败"}), 401

# 保护路由（需JWT认证）
@app.route("/api/protected", methods=["GET"])
@jwt_required()
def protected():
    current_user = get_jwt_identity()
    return jsonify({"user": current_user, "msg": "访问成功"}), 200

from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()

class Student(db.Model):
    id = db.Column(db.String(20), primary_key=True)  # 学号
    name = db.Column(db.String(50), nullable=False)
    class_id = db.Column(db.String(20), db.ForeignKey('class.id'))
    
class Course(db.Model):
    id = db.Column(db.String(20), primary_key=True)  # 课程号
    name = db.Column(db.String(100), nullable=False)
    credits = db.Column(db.Integer, default=0)
    
class Grade(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    student_id = db.Column(db.String(20), db.ForeignKey('student.id'))
    course_id = db.Column(db.String(20), db.ForeignKey('course.id'))
    score = db.Column(db.Float, nullable=False)
    semester = db.Column(db.String(20), nullable=False)

    def calculate_gpa(grades):
    total_credits = 0
    total_weighted_score = 0
    for grade in grades:
        credit = grade.course.credits
        score = grade.score
        # 分数转GPA（标准4.0制）
        if score >= 90:
            gpa = 4.0
        elif score >= 85:
            gpa = 3.7
        # 省略中间分数段...
        else:
            gpa = 0.0
        total_credits += credit
        total_weighted_score += gpa * credit
    return total_weighted_score / total_credits if total_credits > 0 else 0
