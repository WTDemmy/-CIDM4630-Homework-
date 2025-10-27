-- AddCourseEnrollment procedure
DELIMITER //
CREATE PROCEDURE AddCourseEnrollment(
    IN inputStudentID INT,
    IN inputCourseID VARCHAR(10),
    IN inputSemester VARCHAR(20)
)
BEGIN
    INSERT INTO Enrollment(StudentID, CourseID, Semester)
    VALUES (inputStudentID, inputCourseID, inputSemester);
END //
DELIMITER ;

-- DropCourseEnrollment procedure
DELIMITER //
CREATE PROCEDURE DropCourseEnrollment(
    IN inputStudentID INT,
    IN inputCourseID VARCHAR(10),
    IN inputSemester VARCHAR(20)
)
BEGIN
    DELETE FROM Enrollment
    WHERE StudentID = inputStudentID AND CourseID = inputCourseID AND Semester = inputSemester;
END //
DELIMITER ;
